# The Airbnb Effect: A Cross-City ARIMAX Analysis of Short-Term Rentals and Housing Markets

## Overview

This project investigates the causal relationship between the growth of Airbnb listings and residential rental prices in four cities — Milan, Rome, Florence, and New York — spanning the period 2023 to 2025. The analysis is conducted as part of the Group Project for the Digital Economy course and addresses a central policy question at the intersection of platform economics, urban housing markets, and digital regulation.

The core hypothesis is that the expansion of short-term rental platforms generates measurable upward pressure on long-term rental prices by withdrawing residential units from the traditional housing stock. By framing Airbnb listing volume as an exogenous variable, this project uses ARIMAX (Autoregressive Integrated Moving Average with Exogenous Variables) models to quantify this effect with statistical rigour and to compare how different institutional and regulatory environments moderate it.

## Research Question

To what extent does the volume of active Airbnb listings causally affect residential rental prices, and does this effect differ across cities with distinct market structures, densities, and regulatory regimes?

## Analytical Pipeline

### Phase 1 — Data Gathering and Preprocessing

The analysis requires two parallel data streams aligned at a common frequency from 2023 to 2025.

The endogenous variable — the residential rental price series — is sourced from Zillow's ZORI (Zillow Observed Rent Index, unadjusted) for New York, and from OMI (Osservatorio del Mercato Immobiliare, Agenzia delle Entrate) semi-annual quotations for Rome, Milan, and Florence. OMI data provide compravendita and locazione prices per m² at the zona level and are linearly interpolated to produce quarterly estimates aligned with the Airbnb snapshot frequency.

The exogenous variable — the Airbnb market pressure signal — is constructed from Inside Airbnb historical snapshots. For each city and each time period, the number of active entire-home listings is computed per neighbourhood, excluding private rooms and shared rooms on the grounds that only entire-unit withdrawals materially reduce the supply available to long-term tenants.

Spatial alignment is performed via Python to assign each listing to its reference zone: neighbourhood for New York (using the Inside Airbnb `neighbourhood_cleansed` field), and OMI zona for the Italian cities (via `LinkZona`). This allows the analysis to operate at neighbourhood level rather than city aggregate, capturing the heterogeneous geography of the Airbnb effect.

### Phase 2 — ARIMAX Modelling

Before training any model, the stationarity of each rental price series is verified using the Augmented Dickey-Fuller (ADF) test. Non-stationary series are differenced as required. For each city, and for the most data-rich neighbourhoods within each city, a separate ARIMAX model is estimated where the dependent variable is the rental price and the regressors include lagged listing volume and, where applicable, seasonal dummies.

The primary output of each model is the coefficient on the exogenous variable, which represents the estimated change in quarterly rent per m² associated with an incremental change in active Airbnb listings in that zone. This coefficient is the main object of cross-city comparison.

### Phase 3 — Intra-National Comparison (Italy)

The Italian case contrasts three cities representing distinct typologies of tourism-driven urban pressure.

Rome is characterised by pervasive historical tourism with strong seasonal concentration and a geographically diffuse Airbnb penetration centred on the historic centre and Trastevere. Milan exhibits a more business-driven demand structure, with Airbnb presence concentrated around design, fashion, and trade fair events, and gentrification pressure concentrated in specific redeveloped neighbourhoods such as Isola and Porta Nuova. Florence represents the most tourism-saturated case in relative terms: a compact historic centre with a very high ratio of short-term to long-term rental units, subject to ongoing municipal regulation attempts.

The intra-national comparison tests whether the ARIMAX coefficient is larger in cities where Airbnb penetration relative to the residential stock is higher, and whether tourist typology (mass heritage tourism vs. business tourism) moderates the magnitude and seasonality of the effect.

### Phase 4 — International Comparison (Italy vs. New York)

The cross-national comparison assesses whether Italian and US markets differ in the speed and magnitude with which rental prices respond to Airbnb volume shocks. The hypothesis is that the US market, characterised by more liquid rental contracts and mark-to-market repricing, will exhibit higher and faster-loading ARIMAX coefficients than Italian markets, where 4+4 year lease contracts introduce substantial price stickiness.

A secondary objective is the analysis of New York City Local Law 18, enacted in September 2023, which imposed strict registration requirements for short-term rental hosts and caused a sharp and measurable contraction in active Airbnb listings. The ARIMAX framework allows testing whether this exogenous regulatory shock translated into a deceleration of rental price growth, providing a natural quasi-experiment for evaluating the effectiveness of quantity-based short-term rental regulation.

### Phase 5 — Policy Simulation and Recommendations

The final output is a scenario simulation module. Given the estimated ARIMAX coefficients, the model projects the expected rental price trajectory under counterfactual policy scenarios — for example, a 20% cap on active listings in Florence's historic centre, a registration requirement analogous to Local Law 18 applied to Milan, or a reversal of New York's regulatory tightening. Where neighbourhood-level data are sufficient, recommendations are targeted at specific sub-city zones so as to limit externalities on tourism revenue while protecting residential affordability for long-term tenants.

## Data Sources

### Airbnb listing data

| City | 2023–2024 | 2025 | Snapshots |
|------|-----------|------|-----------|
| Rome | `airbnb.csv` — 4 quarterly snapshots (Mar–Dec 2024) | Inside Airbnb — 4 snapshots (Mar, Jun, Jul, Sep) | 8 |
| Milan | `airbnb.csv` — 4 quarterly snapshots (Mar–Dec 2024) | Inside Airbnb — 2 snapshots (Jun, Sep) | 6 |
| Florence | `airbnb.csv` — 4 quarterly snapshots (Mar–Dec 2024) | Inside Airbnb — 3 snapshots (Mar, Jun, Sep) | 7 |
| New York | Kaggle archives — 2 snapshots (Mar 2023, Jan 2024) | Inside Airbnb — 9 snapshots (Mar–Dec 2025) | 11 |

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
  raw_2023_2024/
    airbnb.csv              Italy (Rome, Milan, Florence) — quarterly snapshots 2024
    NYairbnb2023.zip        New York — single snapshot ~March 2023
    NYairbnb2024.zip        New York — single snapshot ~January 2024
    new_york_data.csv       Zillow ZORI rental index for New York
    OMI/                    OMI semiannual quotations (VALORI + ZONE) per city and semester
  raw_2025/
    Roma/                   4 Inside Airbnb snapshots (Mar, Jun, Jul, Sep 2025)
    Milano/                 2 Inside Airbnb snapshots (Jun, Sep 2025)
    Firenze/                3 Inside Airbnb snapshots (Mar, Jun, Sep 2025)
    NewYork/                9 Inside Airbnb snapshots (Mar–Dec 2025)
  processed/
    roma.csv                All Rome Airbnb snapshots merged (2024–2025)
    milano.csv              All Milan Airbnb snapshots merged (2024–2025)
    firenze.csv             All Florence Airbnb snapshots merged (2024–2025)
    new_york.csv            All New York Airbnb snapshots merged (2023–2025)
    master_airbnb.csv       All cities combined — Entire home/apt only
geo/
  neighbourhoodsMilano.geojson
  nieghborhoddsRoma.geojson
  neighbourhoodsNY.geojson
  airbnb_italy_city_neighbourhoods_geojson.geojson
notebook_airbnb.ipynb       Main processing and analysis notebook
```
