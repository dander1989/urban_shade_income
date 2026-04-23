# St. Louis Tree Canopy & Income Equity Analysis

## What This Project Is About

St. Louis is one of the most well-documented examples of redlining in the United States. Starting in the 1930s, the Home Owners' Loan Corporation (HOLC) graded city neighborhoods by "investment risk," and those grades mapped almost perfectly onto race. Neighborhoods that received the lowest grades were systematically denied mortgage lending and public investment for decades.

Urban tree canopy tends to follow historical investment patterns with a significant lag. Trees take decades to grow, which means neighborhoods that experienced disinvestment in the mid-20th century often still show the effects today in the form of lower canopy coverage. This project asks a simple question: does that pattern hold in St. Louis?

Using Census tract-level median household income and NLCD tree canopy raster data, this analysis identifies **priority zones** — tracts with both low income and low canopy coverage — to help guide community-led planting initiatives.

---

## Data Sources

| Dataset | Source | Notes |
|---|---|---|
| Median Household Income (Table B19013, 2020–2024 ACS 5-Year Estimates) | IPUMS NHGIS | Pulled programmatically via `ipumspy` |
| NLCD Tree Canopy Cover (30m raster) | Multi-Resolution Land Characteristics (MRLC) Consortium | Clipped to St. Louis City bounding box |

---

## Tools & Libraries

- Python (Jupyter Notebook)
- `ipumspy` — Census data extraction via NHGIS API
- `geopandas` — vector data handling
- `rasterio` — raster data handling
- `rasterstats` — zonal statistics
- `matplotlib` / `seaborn` — exploratory visualization
- `scipy` — Spearman correlation

---

## Workflow Overview

1. Fetch St. Louis City census tract geometries and median household income via NHGIS API (state FIPS 29, county FIPS 510)
2. Clip NLCD tree canopy raster to St. Louis City bounding box
3. Run zonal statistics to calculate mean canopy percentage per census tract
4. Merge canopy statistics with Census income data
5. Exploratory analysis: null checks, summary statistics, scatterplot, Spearman correlation
6. Set classification thresholds informed by data distributions
7. Apply binary flags and identify priority zones where low income and low canopy overlap
8. Visualize results

---

## Key Methodological Notes

**Why St. Louis?** St. Louis City and St. Louis County are separate FIPS entities (510 and 189 respectively), which required careful filtering to avoid mixing geographies. The city's well-documented history of redlining and neighborhood disinvestment makes it a compelling case study for this kind of analysis.

**Why Spearman correlation?** Income distributions in urban areas tend to be right-skewed, with a few high-income tracts pulling the tail. Spearman's rank-based approach is more robust to that kind of skew and to outliers than Pearson correlation.

**Raster boundary pixels:** Zonal statistics were run with `all_touched=False`, meaning only pixels whose center falls within a tract boundary are included. This avoids inflating or deflating canopy estimates for small tracts by pulling in pixels that straddle boundaries.

**Income threshold:** The citywide median household income for St. Louis City is approximately $56,160 (2020–2024 ACS). Tracts falling meaningfully below this figure are classified as low income in a local context. Thresholds were informed by exploratory data analysis rather than set arbitrarily in advance.

**FEMA / temporal limitations:** This analysis reflects a snapshot in time. Census income data covers 2020–2024 and NLCD canopy data has its own collection date. The two datasets were not collected simultaneously, which is a limitation worth keeping in mind when interpreting results.

---

## Limitations

- NLCD canopy data has a 30-meter resolution, which may not capture fine-grained variation within dense urban tracts
- Median household income is an imperfect proxy for historical disinvestment; neighborhoods undergoing gentrification may show rising incomes while canopy remains low from decades of neglect
- Census tract boundaries do not always align neatly with neighborhood identities or historical redlining boundaries

---

## Repository Structure

```
├── notebook.ipynb        # Main analysis notebook
├── data/
│   └── stl_tracts.geojson  # Cleaned output GeoJSON
├── README.md
```

---

## Author

Built as a geospatial portfolio project to demonstrate Python-based spatial analysis using real-world Census and remote sensing data.
