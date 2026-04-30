# St. Louis Tree Canopy & Income Equity Analysis

## What This Project Is About

St. Louis is one of the most well-documented examples of redlining in the United States. Starting in the 1930s, the Home Owners' Loan Corporation (HOLC) graded city neighborhoods by "investment risk," and those grades mapped almost perfectly onto race. Neighborhoods that received the lowest grades were systematically denied mortgage lending and public investment for decades.

Urban tree canopy tends to follow historical investment patterns with a significant lag. Trees take decades to grow, which means neighborhoods that experienced disinvestment in the mid-20th century often still show the effects today in the form of lower canopy coverage. This project asks a simple question: does that pattern hold in St. Louis?

Using Census tract-level median household income and NLCD tree canopy raster data, this analysis identifies **priority zones** — tracts with both low income and low canopy coverage — to help guide community-led planting initiatives.

---

## Project Outcome

This analysis identified 4 priority zones across 103 census tracts in St. Louis City where both low median household income (below $30,735) and low tree canopy coverage (below 6.45%) overlap. These tracts represent the most actionable targets for community-led planting initiatives, where green infrastructure investment would address both environmental and equity needs simultaneously.

Notably, the Spearman correlation between median income and canopy coverage was weak (r = 0.137, p = 0.167), suggesting that low income and low canopy largely exist in different parts of the city rather than clustering together. This is an honest finding rather than a failure — it means the two conditions of concern are widespread but distributed separately across St. Louis, and the priority zone map identifies the specific places where they do converge.

---

## Learning Outcomes
 
This project demonstrates the following technical and analytical skills:
 
- **Programmatic data acquisition** via the IPUMS NHGIS API (`ipumspy`) and US Census TIGER/Line boundaries (`pygris`), making the analysis fully reproducible without manual data downloads
- **Raster data handling** with `rasterio`, including reading, clipping to a bounding box, and managing coordinate reference systems and nodata values
- **Zonal statistics** using `rasterstats` to summarize raster pixel values within vector polygon boundaries
- **Spatial joins and CRS management** across datasets with mismatched coordinate reference systems
- **Exploratory data analysis** including distribution analysis, skewness assessment, and histogram-based threshold selection
- **Statistical correlation** using Spearman's rank correlation, including appropriate method selection for skewed distributions and honest interpretation of a weak result
- **Data cleaning** including handling Census sentinel values, resolving data type mismatches between joined datasets, and trimming unnecessary columns
- **Thematic cartography** producing choropleth maps, binary classification maps, and a combined categorical map using `geopandas` and `matplotlib`
- **Reproducible research practices** including environment management, API key security via environment variables, and documented methodological decisions throughout the notebook

---

## How to Reproduce

### Prerequisites
- [Miniconda](https://docs.conda.io/en/latest/miniconda.html) or Anaconda installed
- An [IPUMS NHGIS API key](https://account.ipums.org/api_keys) (free to register)
- The [NLCD 2023 Tree Canopy Cover raster](https://www.mrlc.gov/viewer/) clipped to the St. Louis area, saved to `data/raw/`

### Setup
 
1. Clone the repository:
   ```bash
   git clone https://github.com/dander1989/urban_shade_income.git
   cd urban_shade_income
   ```
 
2. Create and activate the Conda environment:
   ```bash
   conda env create -f environment.yml
   conda activate stl_canopy
   ```
 
3. Store your IPUMS API key as an environment variable. Follow the [Conda documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#setting-environment-variables) for your operating system:
   ```bash
   # Example for Linux/Mac
   conda env config vars set IPUMS_API_KEY=your_key_here
   conda activate stl_canopy
   ```
 
4. Download the NLCD 2023 Tree Canopy Cover raster from the [MRLC Viewer](https://www.mrlc.gov/viewer/) using the bounding box coordinates for St. Louis City:
   - West: -90.320515
   - South: 38.532012
   - East: -90.166271
   - North: 38.774368
   Save the downloaded file to `data/raw/` and update the `nlcd_raster_raw` variable in the notebook to match the filename.
5. Open and run the notebook:
   ```bash
   jupyter lab notebook.ipynb
   ```
 
> **Note:** The `data/raw/` folder is excluded from this repository due to file size and licensing. All other data is fetched programmatically within the notebook.

---

## Data Sources

| Dataset | Source | Notes |
|---|---|---|
| Median Household Income (Table B19013, 2020–2024 ACS 5-Year Estimates) | IPUMS NHGIS | Pulled programmatically via `ipumspy` |
| NLCD Tree Canopy Cover (30m raster) | Multi-Resolution Land Characteristics (MRLC) Consortium | Clipped to St. Louis City bounding box |
| St. Louis City Census Tracts (2020) | United States Census Bureau | Pulled programmatically via `pygris` |

---

## Tools & Libraries

- Python (Jupyter Notebook)
- `ipumspy` — Census data extraction via NHGIS API
- `geopandas` — Vector data handling
- `rasterio` — Raster data handling
- `rasterstats` — Zonal statistics
- `matplotlib` / `seaborn` — Exploratory visualization
- `scipy` — Spearman correlation
- `pygris` — TIGER/Line shapefile extraction via US Census Bureau, directly into Python
- `numpy` — Numerical operations and conditional array logic for flag classification
- `pandas` — Tabular data handling and DataFrame operations
- `zipfile` — Extracting CSV data from compressed NHGIS extract downloads
- `shapely` — Converting tract geometries to GeoJSON-like format for rasterio masking

---

## Workflow Overview

1. Fetch St. Louis City census tract geometries
2. Download median household income via NHGIS API (state FIPS 29, county FIPS 510)
3. Join together census tract geometries and median household income data together. Cleanup blank, unnecessary, and redundant fields.
4. Clip NLCD tree canopy raster to St. Louis City bounding box. Reproject tracts to Albers CRS prior to clipping
5. Run zonal statistics to calculate mean canopy percentage per census tract. Reproject tracts beforehand to match raster CRS
6. Merge canopy statistics with Census income data
7. Exploratory analysis: null checks, summary statistics, scatterplot, Spearman correlation
8. Set classification thresholds after checking data distributions and stats
9. Apply binary flags and identify priority zones where low income and low canopy overlap
10. Visualize results and export final gdf to geoJSON

---

## Key Methodological Notes

**Why St. Louis?** St. Louis City and St. Louis County are separate FIPS entities (510 and 189 respectively), which required careful filtering to avoid mixing geographies. The city's well-documented history of redlining and neighborhood disinvestment makes it a compelling case study for this kind of analysis. 

**Why Spearman correlation?** Income distributions in urban areas tend to be right-skewed, with a few high-income tracts pulling the tail. Spearman's rank-based approach is more robust to that kind of skew and to outliers than Pearson correlation.

**Raster boundary pixels:** Zonal statistics were run with `all_touched=False`, meaning only pixels whose center falls within a tract boundary are included. This avoids inflating or deflating canopy estimates for small tracts by pulling in pixels that straddle boundaries.

**Income threshold:** For this project an income threshold of $30,735 was used after figuring out one standard deviation below the mean according to the data that was used. Tracts falling meaningfully below this figure are classified as low income in a local context. Thresholds were informed by exploratory data analysis rather than set arbitrarily in advance.

**Canopy threshold:** Canopy threshold was also informed by EDA rather than set arbitrarily. Threshold was set to 6.45% after calculating one standard deviation below the mean.

**Temporal limitations:** This analysis reflects a snapshot in time. Census income data covers 2020–2024 and NLCD canopy data has its own collection date. Census tracts were from 2020. The three datasets were not collected simultaneously, which is a limitation worth keeping in mind when interpreting results.

**Data limitations:** During EDA, discovered one median_income value (one tract) of -666,666,666, which actually is a placeholder for a tract where the data couldn't be computed due to an insufficient number of sample observations.

**Findings:** During EDA, namely the step computing Spearman coefficient it was found that the data only showed weak correlation between median income and tree canopy (spearman: 0.137, p-value: 0.16692). This means that, at least with the city of St. Louis and the data that was used, that low income and low canopy mostly exist in different parts of the city. Knowing this, the priority zone map still has value and usefulness despite the weak correlation.

---

## Limitations

- NLCD canopy data has a 30-meter resolution, which may not capture fine-grained variation within dense urban tracts. It also may misrepresent actual canopy variation compared to a point-level tree inventory. St. Louis maintains a point-level tree inventory that would provide finer granularity for future analysis.
- Median household income is an imperfect proxy for historical disinvestment; neighborhoods undergoing gentrification may show rising incomes while canopy remains low from decades of neglect
- Census tract boundaries do not always align neatly with neighborhood identities or historical redlining boundaries
- NHGIS data and TIGER/Line shapefiles use different geographic identifier formats. Careful inspection is required before joining.
NHGIS extracts include numerous contextual geographic fields unrelated to the analysis, requiring careful column selection to avoid downstream confusion.

---

## Repository Structure

```
├── notebook.ipynb              # Main analysis notebook
├── README.md
├── .gitignore
├── data/
│   └── processed/
│       └── stl_canopy_income.geojson  # Cleaned output GeoJSON
├── figures/
│   ├── median_income_stl.png
│   ├── avg_tree_canopy_stl.png
│   └── priority_zones.png
```

---

## Future Enhancements

- **Point-level tree inventory:** St. Louis maintains a GeoJSON tree census covering easement, park, and median trees. Replacing the NLCD raster with this dataset via a count-points-in-polygon approach would provide much finer canopy granularity and likely surface a stronger income-canopy signal.
- **Historical redlining overlay:** Overlaying HOLC redlining grade boundaries from Mapping Inequality would allow a direct visual comparison between 1930s investment grades and current canopy/income conditions, strengthening the project's historical narrative.
- **Interactive web map:** Exporting the priority zone data to Leafmap or Kepler.gl would make the findings more accessible to community organizations and non-technical audiences.
- **Temporal analysis:** Comparing NLCD canopy data across multiple collection years (2011, 2016, 2019, 2021, 2023) would reveal whether the canopy gap between neighborhoods is widening or narrowing over time.
- **Multi-city comparison:** Applying the same methodology to other cities with well-documented redlining histories (Memphis, Detroit, Houston) would test whether the weak income-canopy correlation found in St. Louis is consistent or city-specific.

---

## Data Sources & Citations
 
Manson, S., Schroeder, J., Van Riper, D., Kugler, T., Balk, D., Gutmann, M., Leyk, S., King, E., Sherrill, W., and Ruggles, S. IPUMS National Historical Geographic Information System: Version 19.0 [dataset]. Minneapolis, MN: IPUMS. 2024. http://doi.org/10.18128/D050.V19.0
 
Multi-Resolution Land Characteristics (MRLC) Consortium. National Land Cover Database (NLCD) 2023 Tree Canopy Cover. U.S. Geological Survey. https://www.mrlc.gov
 
Walker, K. (2023). pygris: Download and use US Census TIGER/Line shapefiles in Python. https://walker-data.com/pygris/
 
US Census Bureau. American Community Survey 5-Year Estimates, 2020–2024. Table B19013: Median Household Income in the Past 12 Months. https://www.census.gov/programs-surveys/acs
 