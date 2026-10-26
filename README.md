# Heat-Attributable CVD Mortality — South & Southeast Asia [Under Review]

## Overview

This repository contains R code for estimating heat-attributable cardiovascular disease (CVD) mortality across 18 countries in South and Southeast Asia under the SSP5-8.5 (high emissions) scenario. The analysis uses a Monte Carlo simulation framework with age-stratified relative risks (RR) by Köppen climate zone, gridded population projections, and a three-component decomposition to separate the contributions of heat, population growth, and demographic aging to future CVD burden.

---

## Countries Included

**South Asia (7):** Afghanistan (AFG), Bangladesh (BGD), Bhutan (BTN), India (IND), Nepal (NPL), Pakistan (PAK), Sri Lanka (LKA)

**Southeast Asia (11):** Brunei (BRN), Cambodia (KHM), Indonesia (IDN), Laos (LAO), Malaysia (MYS), Myanmar (MMR), Philippines (PHL), Singapore (SGP), Thailand (THA), Timor-Leste (TLS), Vietnam (VNM)

---

## Methods Summary

### 1. Population Attributable Fraction (PAF)

Heat-attributable deaths at each pixel are estimated using:
Relative risks are **age-stratified** by Köppen climate zone:
- Ages 0–64 (`0_39`, `40_59`, `60_69`): `grid_rr_65less.csv`
- Ages 65+ (`70_79`, `80p`): `grid_rr_65plus.csv`

### 2. Monte Carlo Simulation (S = 1000)

For each year, S draws are taken from:
- Lognormal distribution of country-age baseline CVD deaths (parameterised from 95% uncertainty intervals)
- Normal distribution of pixel-level log(RR) per 1°C

Pixel-level 2020 baseline deaths are downscaled from country totals using 2020 population weights.

### 3. Decomposition Analysis

Total burden is decomposed into three additive components using three model variants run simultaneously per draw:

| Variant | Population | Heat | Purpose |
|---------|-----------|------|---------|
| A — Full model | Future pop (with aging) | Future | Main result |
| B — Heat only | Fixed at 2020 | Future | Heat effect alone |
| C — Growth only | Future total at 2020 age mix | Future | Growth without aging |

**Decomposition:**
- **Heat** = B
- **Population growth** = C − B
- **Demographic aging** = A − C
- Heat + Growth + Aging = A (by construction)

`pop_C` is constructed as: future total pixel population × 2020 age fractions, isolating growth from aging.

---

## Input Files

| File | Location | Description |
|------|----------|-------------|
| `pixel_pop_hybrid_Median_SSP5.csv` | `tables-rev1/rev1-pop-ssp/` | Gridded population by age × year (SSP5) |
| `cvd-deaths-iso3.csv` | `asia-gee/cvd-deaths/` | Country-age 2020 CVD deaths with 95% UI |
| `grid_rr_65less.csv` | `tables-rev1/rev1-rr-age-climate-mc-input/` | Pixel RR (under-65), cols: `grid_id`, `mu_logrr`, `se_logrr` |
| `grid_rr_65plus.csv` | `tables-rev1/rev1-rr-age-climate-mc-input/` | Pixel RR (over-65), same columns |
| `OBSERVED_avg_heat_excess_2020.csv` | `asia-gee/tables/` | Observed 2020 heat excess by pixel |
| `ensemble-ssp585-heat-excess.csv` | `asia-gee/tables/` | SSP5-8.5 projected heat excess 2030–2080 |

---

## Script Structure

The main script (`rev1-MC-ssp585-primary.R`) is organised into 13 numbered sections:

| Section | Description |
|---------|-------------|
| 0 | Helper function: lognormal parameters from 95% UI |
| 1 | Load and reshape pixel population (wide → long) |
| 2 | Load country-age deaths; compute lognormal parameters |
| 3 | Load age-stratified RR files (under-65, over-65) |
| 4 | Load and combine heat excess (observed 2020 + scenario 2030+) |
| 5 | Compute 2020 pixel weights for spatial downscaling |
| 6 | Build master model table (`baseY`); construct `pop_C` for Variant C |
| 7 | **Monte Carlo loop** — three variants per year; write pixel-level outputs |
| 8 | Pixel-level attributable rates per 100k by age group (Variant A) |
| 9 | Pixel-level all-ages combined rates per 100k (Variant A) |
| 10 | Country-level summaries: deaths, rates, % change vs 2020 |
| 11 | Country-level deaths tables (long and wide format) |
| 12 | Region-level summaries (South Asia / Southeast Asia) |
| 13 | Decomposition summaries at country and region level |

---

## Output Files

All outputs are written to `mc_outputs/ssp585/`.

### Per-year pixel-level outputs (one file per modelled year)
| File | Contents |
|------|----------|
| `heat_attr_deaths_{year}.csv` | Primary attributable deaths by pixel-age (mean, lo, hi) |
| `heat_attr_rates_100k_{year}.csv` | Attributable rate per 100k by pixel-age |
| `heat_attr_rates_100k_ALL_{year}.csv` | All-ages attributable rate per 100k by pixel |
| `decomp_{year}.csv` | Decomposition deaths + rates for all three components |

### Country-level summary outputs
| File | Contents |
|------|----------|
| `country_rate_by_age.csv` | Rate per 100k by country × age group (wide) |
| `country_rate_all_ages.csv` | All-ages rate per 100k by country (wide) |
| `country_pct_change_by_age_vs_2020.csv` | % rate change vs 2020, by age group |
| `country_pct_change_all_ages_vs_2020.csv` | % rate change vs 2020, all ages |
| `country_deaths_by_age_year.csv` | Absolute deaths by country × age × year |
| `country_deaths_all_ages_year.csv` | Absolute deaths by country × year |
| `country_deaths_by_age_wide.csv` | Deaths wide format, by age |
| `country_deaths_all_ages_wide.csv` | Deaths wide format, all ages |
| `country_decomp_by_age_year.csv` | Decomposition by country × age × year |
| `country_decomp_all_ages_year.csv` | Decomposition by country × year |

### Region-level summary outputs
| File | Contents |
|------|----------|
| `region_deaths_rates_by_age_year.csv` | Regional deaths + rates by age × year |
| `region_deaths_rates_all_ages_year.csv` | Regional deaths + rates, all ages |
| `region_pct_change_by_age_vs_2020.csv` | Regional % rate change by age |
| `region_pct_change_all_ages_vs_2020.csv` | Regional % rate change, all ages |
| `region_rate_by_age_wide.csv` | Regional rates wide format, by age |
| `region_rate_all_ages_wide.csv` | Regional rates wide format, all ages |
| `region_decomp_all_ages_year.csv` | Regional decomposition, all ages |
| `region_decomp_by_age_year.csv` | Regional decomposition by age group |

---

## Modelled Years

2020 (baseline), 2030, 2035, 2040, 2045, 2050, 2055, 2060, 2065, 2070, 2075, 2080

---

## Dependencies

```r
library(pacman)
p_load("data.table", "stringr", "tidyverse", "tidylog")
```

---

## Key Settings

```r
set.seed(1974)   # Reproducibility
S <- 1000        # Monte Carlo iterations
```

---

## Uncertainty

All outputs report **mean**, **2.5th percentile (lo)** and **97.5th percentile (hi)** across the S = 1000 Monte Carlo draws, propagating uncertainty from both baseline death counts and relative risk estimates.

---

## Citation

*[Add citation when published]*

## Contact

Salil V Deo — svd14@case.edu
