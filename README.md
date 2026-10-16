# Caregiver Activity Spaces — NHTS 2017 Microdata

How does caring for young children shape the geography of an adult's day? This project measures the **daily activity space** of U.S. adults (the area that encloses home and every place they visited on the survey day) and compares adults in households with a child aged 0–9 against adults in households with no dependents. It uses the 2017 National Household Travel Survey (NHTS) linked to restricted-use Census block-group geography.

## Research questions

1. Do adults in households with young children have smaller daily activity spaces than comparable adults without dependents?
2. Is that difference concentrated among **women**? Does it differ between weekdays and weekends?
3. How much is it explained by trip frequency, employment, vehicle access, and income?

## Data

| File | Level | Source |
|---|---|---|
| `hhpub.csv`, `perpub.csv`, `trippub.csv` | Household / person / trip | NHTS 2017 public-use files |
| `hhctbg.csv`, `tripctbg.csv` | Home and trip-destination **Census tract + block group** | NHTS 2017 restricted-use geography (data-use agreement) |
| Block-group polygons (2017) | Geography | Census TIGER/Line, downloaded at run time via `tigris` |

**The data are not in this repository.** The block-group geography is restricted-use, and the public files are large. The repository has the code, the NHTS variable dictionaries for the restricted files (`data/*_DICTIONARY.xlsx`), and a knitted report with the full output.

> U.S. Department of Transportation, Federal Highway Administration, 2017 National Household Travel Survey. URL: http://nhts.ornl.gov.

## Method

**Sample construction**
- Households are classified from the person roster plus `YOUNGCHILD` (children under 5 are not on the roster):
  - **with_child**: at least one child aged 0–9.
  - **no_child**: no one aged 0–17. Households with a member aged 10–15 or 70+ are also dropped, so "no_child" means no caregiving dependents at all.
  - Households whose only children are 10–17 are excluded from both groups.
- Respondents aged 23–69. The unit of analysis is the **person-day**.

**Activity-space construction** (minimum convex polygon, following Hasanzadeh et al., 2017)
1. Build 12-digit block-group GEOIDs for each household's home and each trip destination.
2. Download 2017 TIGER/Line block groups for every state in the sample, project them to EPSG:5070 (NAD83 / Conus Albers, equal-area, metres), and take centroids.
3. **Limit location error:** a block-group centroid only stands in well for a real location when the block group is small. Person-days are kept only if the home **and every destination** block group is ≤ 1 km². The size-distribution diagnostics at the end of the report justify this cutoff.
4. Buffer the home centroid by 500 m and each destination centroid by 140 m. Union the buffers, take the convex hull, and record its area in km².

**Models.** OLS on log(activity space, km²) with these controls: age and age², gender, employment, zero-vehicle household, number of trips, number of adults, and income quartile. The sequence adds a weekend effect, child × weekend and child × gender interactions, weekday/weekend stratified models, and a check of how much `n_trips` mediates the child effect. Nested models are compared with F-tests.

## Main results

Analytic sample: **11,289 person-days**. Coefficients are on the log scale; percentages are exp(β) − 1.

| | M1 Unadjusted | M2 Adjusted | M3 + Weekend | M4 + Child × Gender |
|---|---|---|---|---|
| Household has child 0–9 | 0.037 | **−0.094\*\*** | **−0.097\*\*** | −0.007 |
| Child 0–9 × female | | | | **−0.161\*** |
| Weekend | | | −0.164\*\*\* | −0.163\*\*\* |
| Worker | | 0.404\*\*\* | 0.402\*\*\* | 0.394\*\*\* |
| Zero-vehicle household | | −0.405\*\*\* | −0.403\*\*\* | −0.404\*\*\* |
| Trips that day | | 0.186\*\*\* | 0.185\*\*\* | 0.185\*\*\* |
| R² | 0.000 | 0.122 | 0.124 | 0.125 |

\* p<0.05, \*\* p<0.01, \*\*\* p<0.001

- **The raw comparison hides the effect.** Unadjusted, caregivers' activity spaces look the same size (M1). After controls, having a young child is associated with an activity space about **9% smaller** (M2).
- **The constraint falls on mothers.** With a child × gender interaction (M4), the child effect for men is essentially zero. For women it is about **−0.17 log points (≈ 15% smaller)**.
- **No separate weekday/weekend difference.** Weekend activity spaces are about 15% smaller for everyone, but a child × weekend interaction does not improve fit (F-test p = 0.66).
- Employment (+50%), vehicle access (zero-vehicle −33%), and trip count (+20% per trip) are the strongest predictors overall.

**Limitations.** The ≤ 1 km² rule keeps about 9% of person-days, so results describe dense, mostly urban areas. Each activity space is one survey day, not a habitual pattern. The models are cross-sectional associations, not causal estimates.

## Repository contents

| File | Description |
|---|---|
| `01_activity_space_centroid.Rmd` | **Main analysis**: sample construction, centroid-based activity spaces, all models, figures, and block-group size diagnostics |
| `01_activity_space_centroid.html` | Knitted report with all output. Download it and open it in a browser; GitHub shows HTML as source. |
| `02_activity_space_polygon_sensitivity.Rmd` | **Sensitivity analysis**: activity spaces from buffered block-group **polygons** instead of centroids. Restricted to large urbanized areas with heavy rail, fully geocoded person-days, and block groups ≤ 5 km². Standard errors are clustered by household. |
| `data/*_DICTIONARY.xlsx` | Variable dictionaries for the restricted block-group files |
| `data/Citation.docx` | NHTS citation guidance |

## How to run

1. Put the five NHTS CSVs listed above in `data/`.
2. Open `CaregiverActivitySpace.Rproj` in RStudio. Paths are resolved with `here::here()`.
3. Install the packages:
   ```r
   install.packages(c("tidyverse", "sf", "tigris", "here", "broom",
                      "sandwich", "lmtest", "ggeffects", "stargazer"))
   ```
4. Knit `01_activity_space_centroid.Rmd`. The first run downloads TIGER/Line block groups for every state in the sample, and `tigris` caches them for later runs.

## Author

Dawon Oh: study design, data processing, spatial analysis, and modeling.
