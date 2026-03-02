# The Airbnb Effect: A Cross-City ARIMAX Analysis of Short-Term Rentals and Housing Markets

## Overview

This project investigates the causal relationship between the growth of Airbnb listings and residential rental prices in four major cities — Milan, Rome, New York, and Los Angeles — spanning the period 2023 to 2025. The analysis is conducted as part of the Group Project for the Digital Economy course and addresses a central policy question at the intersection of platform economics, urban housing markets, and digital regulation.

The core hypothesis is that the expansion of short-term rental platforms generates measurable upward pressure on long-term rental prices by withdrawing residential units from the traditional housing stock. By framing Airbnb listing volume as an exogenous variable, this project uses ARIMAX (Autoregressive Integrated Moving Average with Exogenous Variables) models to quantify this effect with statistical rigour and to compare how different institutional and regulatory environments moderate it.

## Research Question

To what extent does the volume of active Airbnb listings causally affect residential rental prices, and does this effect differ across cities with distinct market structures, densities, and regulatory regimes?

## Analytical Pipeline

### Phase 1 — Data Gathering and Monthly Preprocessing

The analysis requires two parallel data streams aligned at monthly frequency from 2023 to 2025.

The endogenous variable — the rental price series to be modelled — is sourced from Zillow's ZORI (Zillow Observed Rent Index, unadjusted) for New York and Los Angeles, and from OMI (Osservatorio del Mercato Immobiliare, Agenzia delle Entrate) semi-annual quotations for Rome and Milan, linearly interpolated to produce monthly or quarterly estimates.

The exogenous variable — the Airbnb market pressure signal — is constructed from Inside Airbnb historical snapshots. For each city and each time period, the number of active entire-home listings is computed, excluding shared rooms and private rooms on the grounds that only entire-unit withdrawals materially reduce the supply available to long-term tenants.

Spatial alignment is performed via Python to assign each listing to its reference zone: ZIP Code for US cities, OMI zona for Italian cities. This allows the analysis to operate at neighbourhood level rather than city aggregate, capturing the heterogeneous geography of the Airbnb effect.

### Phase 2 — ARIMAX Modelling

Before training any model, the stationarity of each rental price series is verified using the Augmented Dickey-Fuller (ADF) test. Non-stationary series are differenced as required. For each city, and for the most data-rich neighbourhoods within each city, a separate ARIMAX model is estimated where the dependent variable is the rental price and the regressors include lagged listing volume and, where applicable, seasonal dummies.

The primary output of each model is the coefficient on the exogenous variable, which represents the estimated change in monthly rent associated with an incremental change in active Airbnb listings. This coefficient is the main object of cross-city comparison.

### Phase 3 — Intra-National Comparison

The Italian case contrasts Rome and Milan as two distinct typologies of tourist-driven urban pressure. Rome is characterised by pervasive historical tourism with strong seasonal concentration and a geographically diffuse Airbnb penetration centred on the historic centre. Milan exhibits a more business-driven demand structure, with Airbnb presence concentrated around design, fashion, and trade fair events, and with gentrification pressure concentrated in specific redeveloped neighbourhoods such as Isola and Porta Nuova.

The US case contrasts New York with Los Angeles along the dimension of urban density and regulatory response. The passage of New York City Local Law 18 in September 2023, which imposed strict registration requirements for short-term rental hosts, caused a sharp contraction in active Airbnb listings. The ARIMAX framework allows testing whether this exogenous policy shock translated into a measurable deceleration of rental price growth, or whether rents remained sticky due to structural supply constraints.

### Phase 4 — International Comparison

The cross-national comparison assesses whether Italian and US markets differ in the speed and magnitude with which rental prices respond to Airbnb volume shocks. The hypothesis is that US markets, characterised by more liquid rental contracts and mark-to-market repricing, will exhibit higher and faster-loading ARIMAX coefficients than Italian markets, where 4+4 year lease contracts introduce substantial price stickiness.

A secondary objective is to assess the comparative impact of regulatory intervention. The Italian regulatory landscape has remained fragmented at the national level throughout the observation period, while several US cities and counties have intervened decisively. The comparison of ARIMAX coefficient trajectories before and after intervention dates provides a natural quasi-experiment for evaluating the effectiveness of quantity-based regulation.

### Phase 5 — Policy Simulation and Recommendations

The final output is a scenario simulation module. Given the estimated ARIMAX coefficients, the model projects the expected rental price trajectory under counterfactual policy scenarios — for example, a 20% reduction in active listings enforced through a registration cap in Milan, or an extension of Local Law 18-type regulation to Los Angeles. Where the neighbourhood-level data are sufficient, recommendations are targeted at specific sub-city zones rather than city-wide policy, so as to limit externalities on tourism revenue while protecting residential affordability for long-term tenants.

## Data Sources

Inside Airbnb historical snapshots are the primary source for listing volumes in all four cities for the 2025 period. The 2023-2024 period for Milan and Rome is covered by a consolidated Kaggle dataset. New York 2023-2024 data are sourced from archived Inside Airbnb releases. Los Angeles 2023-2024 data are currently being located; the 2025 data are available across four Inside Airbnb snapshots.

Boundary and neighbourhood files (GeoJSON) are available for all four cities and will be used for spatial joins and choropleth visualisation.

## Repository Structure

```
data/
  raw_2023_2024/      Historical data for Italy (airbnb.csv) and New York (zip archives)
  raw_2025/
    Milano/           Two Inside Airbnb snapshots
    Roma/             Three Inside Airbnb snapshots
    NewYork/          Eleven Inside Airbnb snapshots
    LosAngeles/       Four Inside Airbnb snapshots
geo/                  GeoJSON boundary files for all four cities
notebook_airbnb.ipynb Main processing and analysis notebook
```
