# The Airbnb Effect: A Cross-City Analysis of Short-Term Rentals and Housing Markets

## Overview

This project investigates the relationship between the growth of Airbnb listings and residential rental prices in four cities — Milan, Rome, Florence, and New York — spanning the period 2023 to 2025. The analysis is conducted as part of the Group Project for the Digital Economy course and addresses a central policy question at the intersection of platform economics, urban housing markets, and digital regulation.

The core hypothesis is that the expansion of short-term rental platforms generates measurable upward pressure on long-term rental prices by withdrawing residential units from the traditional housing stock. The project combines exploratory data analysis with two complementary quasi-experimental approaches: an Interrupted Time Series (ITS) model for New York — exploiting the sharp listing contraction caused by Local Law 18 (September 2023) — and a cross-sectional OLS model for the Italian cities, operating at the OMI zona level.

## Research Question

To what extent does the volume of active Airbnb listings affect residential rental prices, and does this effect differ across cities with distinct market structures, regulatory regimes, and tourism intensities?

## Analytical Pipeline

### Phase 1 — Data Gathering and Preprocessing

The analysis requires two parallel data streams aligned at a common frequency from 2023 to 2025.

The residential rental price series is sourced from Zillow's ZORI (Zillow Observed Rent Index, unadjusted) for New York, and from OMI (Osservatorio del Mercato Immobiliare, Agenzia delle Entrate) semi-annual quotations for Rome, Milan, and Florence. OMI data provide compravendita and locazione prices per m² at the zona level and are linearly interpolated to produce monthly estimates.

The Airbnb market pressure signal is constructed from Inside Airbnb historical snapshots. For each city and each time period, the number of active entire-home listings is computed per neighbourhood, excluding private rooms and shared rooms on the grounds that only entire-unit withdrawals materially reduce the supply available to long-term tenants. Between snapshot dates, listing counts are held constant via forward-fill to produce a continuous monthly series.

### Phase 2 — Exploratory Data Analysis (EDA)

Before any modelling, a systematic EDA characterises the data along six dimensions:

- **Listing trends over time** — monthly active Entire home/apt listings for all four cities on a common timeline, with real snapshot dates marked. The sharp contraction in New York following Local Law 18 is immediately visible.
- **New York Local Law 18 inspection** — dual-axis chart overlaying listing volume (left axis) and Zillow ZORI rent index (right axis), with the September 2023 enforcement date annotated. This motivates the ITS modelling strategy.
- **OMI rent trends** — city-level canone locazione (€/m²/mese) for Rome, Milan, and Florence across the five available semesters (2023/1–2025/1), with interpolated values shown as dashed lines between real data points.
- **Airbnb price distributions** — box plots of nightly prices clipped at the 95th percentile, shown separately for Italian cities (EUR) and New York (USD) since the two currencies are not directly comparable.
- **Neighbourhood concentration** — top-10 most listed neighbourhoods per city based on the most recent snapshot, revealing the geographic concentration of Airbnb supply.
- **New York seasonality heatmap** — month × year grid of listing counts, making seasonal patterns and the LL18 shock visible simultaneously.

### Phase 3 — New York: Interrupted Time Series (ITS)

New York provides the cleanest identification opportunity: Local Law 18, enforced from September 2023, imposed a host registration requirement that caused active Entire home/apt listings to fall from approximately 24,000 to 11,500 within two months — a ~55% contraction. This constitutes a plausibly exogenous shock to short-term rental supply.

The ITS model estimates the effect of this shock on the Zillow ZORI rent index:

$$\text{ZORI}_t = \alpha + \beta_1 t + \beta_2 \cdot \mathbf{1}_{t > T} + \beta_3 (t - T) \cdot \mathbf{1}_{t > T} + \varepsilon_t$$

where $T$ is September 2023, $\beta_2$ captures an immediate level shift in rents (if any), and $\beta_3$ captures a change in the post-intervention trend. The hypothesis consistent with the supply-withdrawal channel is $\beta_2 > 0$ (rents jump) or $\beta_3 < 0$ (rent growth decelerates following the supply restoration as hosts exit the platform permanently).

With N = 34 monthly observations (March 2023 – December 2025) of real Zillow data, this model has adequate degrees of freedom for a two-breakpoint OLS regression.

### Phase 4 — Italy: Cross-Sectional OLS per OMI zona

For the Italian cities, the time dimension is too thin (5 semesters) to support time-series modelling. Instead, the analysis exploits between-zone variation within each city and across the three cities.

The unit of observation is an **OMI zona × semester** cell. Each zona is matched to the count of active Airbnb listings in the corresponding neighbourhood via a spatial join between the Inside Airbnb `neighbourhood_cleansed` field and the OMI `LinkZona` identifier, using the GeoJSON boundary files available in `geo/`. Listing density is computed as listings per resident population or per km² of the zona.

The model:

$$\text{loc\_medio}_{z,t} = \alpha + \beta \log(1 + \text{density}_{z,t}) + \gamma_f + \delta_c + \varepsilon_{z,t}$$

where $\gamma_f$ is a fascia fixed effect (A/B/C capturing centre vs. periphery) and $\delta_c$ is a city fixed effect. This specification absorbs time-invariant city and zone characteristics and isolates the within-fascia variation in listing density as the identifying source of variation.

The cross-city comparison tests whether $\beta$ is larger for Florence — the most tourism-saturated city in relative terms — than for Rome or Milan, consistent with the supply-withdrawal hypothesis.

## Data Sources

### Airbnb listing data

| City | 2023–2024 | 2025 | Snapshots |
|------|-----------|------|-----------|
| Rome | `airbnb.csv` — 4 quarterly snapshots (Mar–Dec 2024) | Inside Airbnb — 4 snapshots (Mar, Jun, Jul, Sep) | 8 |
| Milan | `airbnb.csv` — 4 quarterly snapshots (Mar–Dec 2024) | Inside Airbnb — 2 snapshots (Jun, Sep) | 6 |
| Florence | `airbnb.csv` — 4 quarterly snapshots (Mar–Dec 2024) | Inside Airbnb — 3 snapshots (Mar, Jun, Sep) | 7 |
| New York | Kaggle archives — 2 snapshots (Mar 2023, Jan 2024) | Inside Airbnb — 9 snapshots (Mar–Sep, Nov, Dec 2025) | 11 |

### Residential price data

| City | Source | Format | Frequency |
|------|--------|--------|-----------|
| Rome | OMI — Agenzia delle Entrate | `QI_*_VALORI.csv` + `QI_*_ZONE.csv` (per semestre) | Semiannual → interpolated |
| Milan | OMI — Agenzia delle Entrate | same | Semiannual → interpolated |
| Florence | OMI — Agenzia delle Entrate | same | Semiannual → interpolated |
| New York | Zillow ZORI / `new_york_data.csv` | wide-format monthly time series | Monthly |

### Boundary files

GeoJSON neighbourhood boundary files are available for all four cities and will be used for spatial joins and choropleth visualisation.

## Repository Structure

```
data/
  airbnb/                             Airbnb listing data (all sources)
    italy_2024/
      airbnb.csv                      Italy (Rome, Milan, Florence) — 4 quarterly snapshots 2024
    new_york_2023_2024/
      NYairbnb2023.zip                New York — single snapshot ~March 2023
      NYairbnb2024.zip                New York — single snapshot ~January 2024
      new_york_data.csv               Zillow ZORI rental index for New York
    insideairbnb_2025/
      Roma/                           Inside Airbnb snapshots (2025)
      Milano/                         Inside Airbnb snapshots (2025)
      Firenze/                        Inside Airbnb snapshots (2025)
      NewYork/                        Inside Airbnb snapshots (2025)
      LosAngeles/                     Inside Airbnb snapshots (2025)
  omi/                                OMI — quotazioni immobiliari (Agenzia delle Entrate)
    Roma/                             VALORI + ZONE: 2023/1, 2023/2, 2024/1, 2024/2, 2025/1
    Milano/                           VALORI + ZONE: 2023/1, 2023/2, 2024/1, 2024/2, 2025/1
    Firenze/                          VALORI + ZONE: 2023/1, 2023/2, 2024/1, 2024/2, 2025/1
  processed/
    roma.csv                          All Rome Airbnb snapshots merged (2024–2025)
    milano.csv                        All Milan Airbnb snapshots merged (2024–2025)
    firenze.csv                       All Florence Airbnb snapshots merged (2024–2025)
    new_york.csv                      All New York Airbnb snapshots merged (2023–2025)
    master_airbnb.csv                 All cities combined — Entire home/apt only
    omi_roma.csv                      OMI abitazioni civili — Roma (tutti i semestri)
    omi_milano.csv                    OMI abitazioni civili — Milano (tutti i semestri)
    omi_firenze.csv                   OMI abitazioni civili — Firenze (tutti i semestri)
    omi_master.csv                    OMI abitazioni civili — tutte le città
    panel_italy.csv                   Panel mensile (OMI €/m²/mese + n_listings) — città italiane — input OLS
    panel_ny.csv                      Panel mensile (ZORI $/mese + n_listings) — New York — input ITS
geo/
  neighbourhoodsMilano.geojson
  nieghborhoddsRoma.geojson
  neighbourhoodsNY.geojson
  neighbourhoodsLA.geojson
  airbnb_italy_city_neighbourhoods_geojson.geojson
plots/
  01_listing_trends.png               Trend listing mensili per città (2023–2025)
  02_ny_ll18_zori.png                 NYC — dual axis: listing + ZORI, con annotazione Local Law 18
  03_omi_trends.png                   Canone locazione OMI per città italiana (semestri reali + interpolato)
  04_price_distribution.png           Distribuzione prezzi Airbnb per città (boxplot, clip 95°p)
  05_top_neighbourhoods.png           Top 10 quartieri per n° listing — snapshot più recente
  06_ny_heatmap.png                   NYC — heatmap listing per anno × mese (stagionalità + LL18)
01_eda_airbnb.ipynb                   Main EDA notebook (Phase 1–2 preprocessing + Phase 2 EDA)
```
