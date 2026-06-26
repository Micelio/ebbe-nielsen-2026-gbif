# GBIF-on-QLever — example SPARQL queries

Endpoint (HTTP API): `https://qlever.dev/api/gbif`
Web UI: `https://ui.qlever.dev/gbif`
Wikidata (for federation, same engine): `https://qlever.dev/api/wikidata`

All queries below were run against the live endpoint.

## Graph facts worth knowing
- **No `skos:prefLabel`.** Names are on `rdfs:label` (canonical, author-less, e.g. `"Oenanthe"`), `skos:altLabel`, and `dwc:scientificName` (with authorship).
- **Linnaean ranks are denormalised as literals on every taxon:** `dwc:kingdom`, `dwc:phylum`, `dwc:class`, `dwc:order`, `dwc:family`, `dwc:genus` — read the kingdom directly, no need to walk `skos:broader+`.
- Hierarchy: `skos:broader` (child → parent); `skos:broader*` for all descendants.
- Occurrence → taxon link: `dwciri:toTaxon`.
- Coordinates: `dwc:decimalLatitude` / `dwc:decimalLongitude`, and `wdt:P625` as a `geo:wktLiteral` — project a `?geometry` bound to `wdt:P625` to trigger QLever's **Map view**.
- Value vocabularies are lowercase IRIs: rank `…/terms/rank/{species,genus,…}`, status `…/terms/status/{accepted,synonym,…}`.

## Common prefixes
```sparql
PREFIX rdfs:   <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos:   <http://www.w3.org/2004/02/skos/core#>
PREFIX gbif:   <https://rs.gbif.org/terms/>
PREFIX rank:   <https://rs.gbif.org/terms/rank/>
PREFIX status: <https://rs.gbif.org/terms/status/>
PREFIX dwc:    <http://rs.tdwg.org/dwc/terms/>
PREFIX dwciri: <http://rs.tdwg.org/dwc/iri/>
PREFIX wdt:    <http://www.wikidata.org/prop/direct/>
```

---

## 1. Cross-kingdom homonyms
Two accepted taxa, same name and rank, but different kingdoms.

```sparql
SELECT ?name ?rank ?kingdomA ?kingdomB ?a ?b WHERE {
  ?a rdfs:label ?name ; gbif:rank ?rank ;
     gbif:taxonomicStatus status:accepted ; dwc:kingdom ?kingdomA .
  ?b rdfs:label ?name ; gbif:rank ?rank ;
     gbif:taxonomicStatus status:accepted ; dwc:kingdom ?kingdomB .
  FILTER(STR(?a) < STR(?b))
  FILTER(STR(?kingdomA) != STR(?kingdomB))
  FILTER(STR(?kingdomA) != "incertae sedis" && STR(?kingdomB) != "incertae sedis")
}
LIMIT 100
```
Result: **1,769 pairs** (1,712 distinct names) — 1,652 genus-rank, 117 species-rank.

## 2. One name, two kingdoms — *Oenanthe* (a bird and a plant)
```sparql
SELECT ?scientificName ?kingdom ?phylum ?taxon WHERE {
  ?taxon rdfs:label "Oenanthe" ;
         gbif:rank rank:genus ;
         gbif:taxonomicStatus status:accepted ;
         dwc:scientificName ?scientificName ;
         dwc:kingdom ?kingdom ;
         dwc:phylum  ?phylum .
}
```
Returns:
- `Oenanthe Vieillot, 1816` — Animalia / Chordata — wheatears (birds) — https://www.gbif.org/species/2492483
- `Oenanthe L.` — Plantae / Tracheophyta — water-dropworts (plants) — https://www.gbif.org/species/3034893

## 3. Count homonyms, overall and by rank
```sparql
SELECT (COUNT(*) AS ?pairs) (COUNT(DISTINCT ?name) AS ?names) WHERE {
  ?a rdfs:label ?name ; gbif:rank ?rank ;
     gbif:taxonomicStatus status:accepted ; dwc:kingdom ?kA .
  ?b rdfs:label ?name ; gbif:rank ?rank ;
     gbif:taxonomicStatus status:accepted ; dwc:kingdom ?kB .
  FILTER(STR(?a) < STR(?b))
  FILTER(STR(?kA) != STR(?kB))
  FILTER(STR(?kA) != "incertae sedis" && STR(?kB) != "incertae sedis")
}
```
Swap the projection for `SELECT ?rank (COUNT(*) AS ?n) … GROUP BY ?rank ORDER BY DESC(?n)` to get the by-rank split (genus 1,652 · species 117).

## 4. Curation gap — homonym taxa already mapped to Wikidata (federated)
How many of the taxa caught in cross-kingdom homonyms already carry a Wikidata identifier (`wdt:P846`).
```sparql
SELECT (COUNT(DISTINCT ?taxon) AS ?mapped) WHERE {
  {
    SELECT DISTINCT ?taxon ?id WHERE {
      ?taxon rdfs:label ?name ; gbif:rank ?rank ;
             gbif:taxonomicStatus status:accepted ; dwc:kingdom ?k .
      ?other rdfs:label ?name ; gbif:rank ?rank ;
             gbif:taxonomicStatus status:accepted ; dwc:kingdom ?k2 .
      FILTER(STR(?k) != STR(?k2))
      FILTER(STR(?k) != "incertae sedis" && STR(?k2) != "incertae sedis")
      BIND(STRAFTER(STR(?taxon),"species/") AS ?id)
    }
  }
  SERVICE <https://qlever.dev/api/wikidata> { ?wd wdt:P846 ?id . }
}
```
Result: of **3,452** homonym taxa, **2,670 (77%)** are already in Wikidata; **782 (23%)** are not — the curation backlog.

## 5. Occurrences on a map — autocomplete a taxon, then "Map view"
Autocomplete the taxon object (the `rdfs:label` value), then click **Map view** in the QLever UI — `?geometry` (bound to `wdt:P625`) triggers the map.
Example: the black widow, *Latrodectus* (54,331 georeferenced occurrences).
Shared query: https://ui.qlever.dev/gbif/n1nIls

```sparql
SELECT ?occurrence ?geometry WHERE {
  ?genus rdfs:label "Latrodectus" ; gbif:rank rank:genus .
  ?species skos:broader ?genus .
  ?occurrence dwciri:toTaxon ?species ;
              wdt:P625 ?geometry .
}
```
For a single species, drop the genus/`skos:broader` lines and match the species label directly, e.g. `?species rdfs:label "Latrodectus mactans" .`

---

## 6. GBIF × OSM — occurrences inside nature reserves (federated, GeoSPARQL)
Occurrences of a taxon that fall **inside OSM protected areas**. **Run on `https://qlever.dev/api/osm-planet`** (not the gbif endpoint): pull the GBIF points in via `SERVICE`, use the reserve **relation's** polygon (`rdf:type osm:relation` + `geo:hasGeometry/geo:asWKT`), and let that *indexed* polygon be the first argument of `geof:sfContains`. ~18 s. Team-validated (Jerven Bolleman; working osm-planet form from Hannah Bast).

> **Why this direction.** Running it on the *gbif* endpoint and injecting the GBIF point into the osm `SERVICE` 500s — there the point isn't index-backed, so QLever can't build the spatial join (`"Geometric relations via geof:sfIntersects … currently only [supported for indexed geometries]"`). Hosting it on osm-planet (polygons local) fixes that.

```sparql
PREFIX gbifsp: <https://www.gbif.org/species/>
PREFIX dwc:    <http://rs.tdwg.org/dwc/terms/>
PREFIX dwciri: <http://rs.tdwg.org/dwc/iri/>
PREFIX wdt:    <http://www.wikidata.org/prop/direct/>
PREFIX geof:   <http://www.opengis.net/def/function/geosparql/>
PREFIX osmkey: <https://www.openstreetmap.org/wiki/Key:>
PREFIX rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX osm:    <https://www.openstreetmap.org/>
PREFIX geo:    <http://www.opengis.net/ont/geosparql#>

SELECT ?occurrence ?country ?date ?geometry WHERE {
  SERVICE <https://qlever.dev/api/gbif> {
    ?occurrence dwciri:toTaxon gbifsp:5219404 ;
                wdt:P625 ?geometry ;
                dwc:basisOfRecord "HUMAN_OBSERVATION" .
    OPTIONAL { ?occurrence dwc:countryCode ?country }
    OPTIONAL { ?occurrence dwc:eventDate   ?date }
  }
  VALUES ?protect_class { "1a" "1" "1b" "2" "3" "4" "5" "6" "7" "97" "98" "99" }
  ?reserve osmkey:protect_class ?protect_class ;
           rdf:type osm:relation ;
           geo:hasGeometry/geo:asWKT ?reserveGeometry .
  FILTER(geof:sfContains(?reserveGeometry, ?geometry))
}
LIMIT 100
```
Notes: `rdf:type osm:relation` selects the reserve as a relation (its full multipolygon), which is index-backed and fast — much quicker than walking `osmway:member` boundaries. For occurrences **outside** reserves, compute total − inside, or `MINUS` this set. Demoed by Tiago Lubiana's tool (tiago.bio.br/gbif_sparql).
