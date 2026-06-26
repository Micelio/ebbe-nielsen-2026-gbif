# 2026 Ebbe Nielsen Challenge — form answers

Copy these into the official form. Verify emails/affiliations before submitting.

## Primary contact name
Andra Waagmeester

## Other team members' names
Hannah Bast; Jerven Bolleman; Tiago Lubiana; Robert Hoehndorf

## Email addresses of all team members
- Andra Waagmeester — andra@waagmeester.net
- Hannah Bast — *<add>*
- Jerven Bolleman — *<add>*
- Tiago Lubiana — *<add>*
- Robert Hoehndorf — *<add>*

## Affiliations (country, institution, ORCID)
- Andra Waagmeester — Micelio, Belgium; Maastricht University, Netherlands — ORCID 0000-0001-9773-4008
- Hannah Bast — University of Freiburg, Germany — ORCID 0000-0003-1213-6776
- Jerven Bolleman — SIB Swiss Institute of Bioinformatics, Switzerland — ORCID 0000-0002-7449-1266
- Tiago Lubiana — University of São Paulo, Brazil — ORCID 0000-0003-2473-2313
- Robert Hoehndorf — KAUST, Saudi Arabia — ORCID 0000-0001-8149-5890

## Submission title
Living up to the promise of Darwin Core RDF — SPARQLing GBIF

## Abstract and rationale (max. 1,000 words)
GBIF is the central, FAIR home for biodiversity occurrence data, built on Darwin Core. Its APIs already move enormous volumes of data — but each integration with another resource is, in effect, a separate "boat trip": one bespoke query at a time. We show that GBIF can instead be a fully bridged member of the web of linked open data, queryable in standard SPARQL and federatable with other knowledge graphs, affordably and at a speed that real users will accept.

We took roughly 261 GB of GBIF occurrence records, in the simple Parquet export, together with the taxonomic backbone, and transformed them into RDF (the `gbif_parquet` converter). We loaded the result into the open-source QLever SPARQL engine with geoSPARQL support, on modest hardware, and exposed a public endpoint (https://qlever.dev/api/gbif) with a query UI (https://ui.qlever.dev/gbif). There is no cloud bill, no quota, and no login. Converting GBIF to RDF has been done before; what is new here is that a standardized, public, geospatial SPARQL interface over the whole of GBIF can be provided cheaply and responsively, and — crucially — federated live with Wikidata and OpenStreetMap.

Why it matters to the GBIF community:

1. **Cross-graph questions that no single dataset can answer.** Because GBIF now speaks RDF, a single federated query can join occurrences with OpenStreetMap and Wikidata. We demonstrate a genuinely spatial conservation question — *which observations of lions (Panthera leo) fall outside any protected area?* — by joining GBIF occurrence points to OpenStreetMap protected-area polygons with GeoSPARQL (`geof:sfContains`). The same pattern answers "inside vs outside reserves" for any taxon. This is the kind of analysis that today requires bespoke pipelines; here it is one query, live.

2. **Data quality and curation feedback.** Querying GBIF by name is ambiguous: identical scientific names governed by different nomenclatural codes (homonyms) collide. Across the backbone we find 1,769 accepted cross-kingdom homonym pairs (e.g. *Oenanthe* — both a bird and a plant). Mapping each GBIF taxon to a stable Wikidata identifier removes the ambiguity; a federated query shows that ~77% of the taxa involved already carry a Wikidata ID, leaving a precise, machine-generated curation backlog. Having GBIF as RDF lets the community both *consume* Wikidata and *improve* it.

3. **Multilinguality and places.** RDF labels can carry many languages at once (Wikidata has labels in 300+ languages for taxa mapped to GBIF), and giving every location its own identifier lets one place carry the names people actually use — even historical localities without coordinates.

4. **Provenance, not flattening.** A record is not a fact about the world; it is a claim by someone. Modelling the epistemic layer separately from the ontological one lets us track provenance and even detect contradicting identifications — important as citizen-science observations (e.g. iNaturalist) feed the system.

How audiences benefit. Researchers and data managers get a free, standards-based, geospatial query surface over GBIF that federates with the wider linked-data web — no infrastructure to run. To make this concrete (and to show how low the barrier now is), we include two small, openly licensed web apps, each a single static HTML page querying the live endpoint directly from the browser, each built with AI assistance in minutes:

- **EU invasive-species spread** (https://www.micelio.be/eu-invasive-spread-gbif/): pick any species on the EU list of Invasive Alien Species of Union concern and watch a year-by-year heatmap of its spread across Europe, with the species' Wikidata photo and its GBIF, iNaturalist and NCBI identifiers.
- **Occurrences in nature reserves** (https://tiago.bio.br/gbif_sparql/, by Tiago Lubiana): pick a taxon and see its GBIF occurrences against OpenStreetMap protected-area boundaries.

These illustrate a broader point for GBIF: once occurrence data is RDF on a public SPARQL endpoint, useful, shareable tools can be built rapidly — even by non-specialists with AI assistance — because the heavy lifting (indexing, geospatial joins, federation) lives in the endpoint, not the app.

In short: GBIF, in RDF, on affordable hardware, openly queryable, geospatial, federated, multilingual and provenance-aware. The community is ready to sail off the island; we show the ship is both seaworthy and cheap to build.

## Operating instructions
1. **Run a query.** Open https://ui.qlever.dev/gbif, paste an example query from the repository (`sparql/examples.md`), and press **Execute**.
2. **Map the results.** Run a query that returns `?geometry` (bound to `wdt:P625`) and click **Map view** to see a heatmap.
3. **Spatial GBIF × OpenStreetMap** (e.g. lions outside protected areas). Open https://ui.qlever.dev/osm-planet, paste the reserve query (repo `sparql/examples.md`, §6), Execute, then Map view. (It runs on the osm-planet endpoint and pulls GBIF points in via SERVICE.)
4. **Invasive-species spread app.** Open https://www.micelio.be/eu-invasive-spread-gbif/, choose a species, press **Load & play**; use the year slider; click a result for its Wikidata photo + identifiers.
5. **Reserve app.** Open https://tiago.bio.br/gbif_sparql/, type a taxon, choose "Nature reserves only", press **Map it**.
6. **Watch the 5-minute film** (link below) for the full walkthrough.

## Link to video / screencast (≤5 min)
https://youtu.be/Wg0VYpE6Cu0 — and the companion screencast https://youtu.be/vKqK2SePcXY

## Link(s) to source location
- Submission repo: https://github.com/Micelio/ebbe-nielsen-2026-gbif
- Spread app: https://github.com/Micelio/eu-invasive-spread-gbif
- Reserve tool: https://github.com/lubianat/gbif_sparql
- Parquet → RDF converter: https://github.com/Micelio/gbif_parquet
- QLever engine: https://github.com/ad-freiburg/qlever
- QLever Petrimaps (Map view): https://github.com/ad-freiburg/qlever-petrimaps
- Live endpoint: https://qlever.dev/api/gbif
