# Rocks, Soil, and Deep Time: Understanding Local Geology Through Public Data

**Domain:** Geospatial Analysis · Environmental Data Integration  
**Tools:** Python, scikit-learn, REST APIs (USDA SSURGO, Macrostrat)  

---

## The Question

Digging in a Central Austin backyard turns up a puzzling mix: soft white chalk, dark glassy chert, caliche nodules, sticky expansive clay, and smooth pink crystalline rocks that look like granite — sometimes all in the same shovelful. What is any of this, and why is there so much variety?

It turns out the neighborhood sits at or near the contact zone between two geological formations deposited in an ancient shallow inland sea 70-90 million years ago: Austin Chalk (limestone, high calcium carbonate, alkaline) and Taylor Marl (clay-rich, darker, more acidic). The pink crystalline rocks are a different story entirely — likely derived from the Llano Uplift, Precambrian igneous and metamorphic formations over a billion years old exposed in the Texas Hill Country.

Soil chemistry reflects the geology underneath. That's the premise of this project.

---

## The Data

Two publicly available datasets, both queried via REST API:

**USDA SSURGO** — the most detailed soil survey database available for the US. Queried for Travis County using the Soil Data Access REST endpoint, joining three tables (map unit, component, horizon) to produce one row per soil depth layer. 384 rows after retrieval, 315 after cleaning non-soil components, bedrock horizons, and paralithic layers.

**Macrostrat** — a geological formation database providing rock unit descriptions, age ranges, lithology, and depositional environment. Queried for two columns: San Marcos Arch (local Cretaceous formations) and Llano Uplift (Precambrian igneous and metamorphic units). 101 geological units spanning from recent surface deposits to rocks over 1.2 billion years old.

The lithology field returns JSON-structured data requiring parsing to extract dominant rock type and rock class — a non-trivial step handled with custom extraction functions.

---

## The Approach

Four questions, each building on the last:

**1. Does depth predict soil pH or calcium carbonate?**  
Simple linear regression. R-squared of 0.006 for pH and 0.002 for calcium carbonate. Depth alone explains essentially nothing — the geology varies too much by location across the county for depth to be a reliable predictor.

**2. Can soil chemistry predict pH?**  
Multiple linear regression using calcium carbonate %, silt %, clay %, organic matter, and depth. Sand % excluded due to multicollinearity (clay + silt + sand = 100%). Cross-validated R-squared of 0.33 — captures some signal, leaves most variation unexplained. The dataset clusters so heavily around pH 8.0-8.2 that there isn't much variation to predict.

**3. Do soils cluster into geologically meaningful groups?**  
K-Means clustering on the same features (k=3 selected over k=4 based on interpretability). Three clusters emerged: limestone-dominated horizons with high calcium carbonate, organic-rich clay surface soils, and deeper subsoil horizons. Clusters validated against soil taxonomy and drainage class — they don't map onto taxonomic orders but do align with geological character. Mean silhouette score of 0.27 indicates weak to moderate separation, consistent with what the scatter plots showed.

**4. Can we predict which geological formation a soil horizon came from?**  
Cluster labels mapped to Macrostrat formations (Austin Chalk, Taylor Marl), then used as training labels for binary logistic regression. Cross-validated accuracy of 98.5%, F1 of 0.977. Calcium carbonate percentage is the dominant predictor. The soil chemistry of these two formations is distinct enough that a simple model can tell them apart with near-perfect reliability.

---

## The Local Test

The county-wide model was applied to a specific neighborhood polygon drawn in Google My Maps and passed to the SSURGO spatial API. Results were geologically coherent: limestone-derived soil series predicted as Austin Chalk, clay-dominated series as Taylor Marl, and the Austin soil series — which sits at the contact zone — split between both formations.

---

## Key Finding

A shovelful of Central Austin dirt can contain material spanning nearly a billion years of geological time. The soil chemistry makes that legible: calcium carbonate alone is sufficient to distinguish Austin Chalk from Taylor Marl with near-perfect accuracy, because the formations are chemically distinct enough that the soil remembers where it came from.

---

## What This Demonstrates

- Cross-domain data integration from two independent public APIs
- Layered analytical approach: regression, clustering, classification, and spatial filtering in sequence
- Honest model evaluation — reporting weak results (Models 1 and 2) alongside strong ones (Model 4)
- Domain-driven interpretation connecting statistical outputs to real geological context
- Self-directed inquiry: question formulation, data sourcing, and analysis design without a predefined problem set
