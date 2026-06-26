# Ebbe Nielsen 2026 — demo storyboard (compiled examples)

Live order for the demo screencasts. Two endpoints:
`gbif` = https://qlever.dev/api/gbif · `osm-planet` = https://qlever.dev/api/osm-planet

## 1. Lions outside protected areas  (GBIF × OpenStreetMap)
- Link: https://qlever.dev/osm-planet/xAc6B9  · run on **osm-planet**
- Do: **run the query, then click Map view.**
- Point: federating GBIF occurrences with OSM protected-area polygons (GeoSPARQL) answers a question neither has alone — which *Panthera leo* records fall **outside** protection (4,193). ~18 s.
- Note: the saved query has a duplicated PREFIX block — clean copy in `GBIF_SPARQL_examples.md` #6.

## 2. Homonyms across kingdoms
- Link: https://ui.qlever.dev/gbif/6Bzk1O  · **gbif**
- Do: run. Shows the cross-kingdom name clashes (1,769 pairs; e.g. genera that are both an animal and a plant).

## 3. (optional) One name, two kingdoms — Oenanthe
- Link: https://ui.qlever.dev/gbif/FbUNNf  · **gbif**  (fixed — has `rdfs:label "Oenanthe"`)
- Do: run. Two rows — Oenanthe the bird (Animalia) and Oenanthe the plant (Plantae).

## 4. Tiago's app — results in an app
- Link: https://tiago.bio.br/gbif_sparql  · screencast youtu.be/vKqK2SePcXY
- Say: *"You can drop these query results straight into an app — and this one was built with AI in about five minutes."*

## 5. The EU list of Invasive Alien Species of Union concern
- Page: https://environment.ec.europa.eu/topics/nature-and-biodiversity/invasive-alien-species_en
- Say: 88+/114 species of Union concern — a real, policy-relevant target list.

## 6. Resolve the EU list against GBIF  (your query)
- Link: https://ui.qlever.dev/gbif/GYNrpL  · **gbif**
- Do: run. 110 EU names → backbone taxon, status, accepted name, and whether GBIF has occurrences (surfaces synonyms + coverage gaps in one query).

## 7. The spread app  (your app)
- Link: https://www.micelio.be/eu-invasive-spread-gbif/
- Do: pick a species (e.g. *Vespa velutina*) → watch the year-by-year spread heatmap; show the Wikidata photo + GBIF/iNaturalist/NCBI identifiers.

## 8. Closing remark
- Say: *"From a SPARQL query to a working, shareable app in minutes — AI makes this kind of rapid app development easy."*

---
### Narrative arc
Queries (1–3) prove the federation → Tiago's app (4) shows results become a usable tool, built with AI in minutes → the EU list (5) gives a real target → your resolver query (6) and spread app (7) close the loop → end on the AI-rapid-development point (8).
