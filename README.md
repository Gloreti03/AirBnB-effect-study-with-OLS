# The Airbnb Effect: A Cross-City Analysis of Short-Term Rentals and Housing Markets

## Overview

This project investigates the relationship between the growth of Airbnb listings and residential rental prices in four cities — Milan, Rome, Florence, and New York — spanning the period 2023 to 2025. The analysis is conducted as part of the Group Project for the Internet and Network Economics course and addresses a central policy question at the intersection of platform economics, urban housing markets, and digital regulation.

The core hypothesis is that the expansion of short-term rental platforms generates measurable upward pressure on long-term rental prices by withdrawing residential units from the traditional housing stock. The project combines exploratory data analysis with two complementary quasi-experimental approaches: an Interrupted Time Series (ITS) model for New York — exploiting the sharp listing contraction caused by Local Law 18 (September 2023) — and a cross-sectional OLS model for the Italian cities, operating at the OMI zona level.

## Research Question

To what extent does the volume of active Airbnb listings affect residential rental prices, and does this effect differ across cities with distinct market structures, regulatory regimes, and tourism intensities?

---

## Analytical Pipeline

### Phase 1 — Data Gathering and Preprocessing

The analysis requires two parallel data streams aligned at a common frequency from 2023 to 2025.

**Residential rental prices** are sourced from:
- **Zillow ZORI** (Zillow Observed Rent Index, unadjusted) for New York — monthly series, expressed in $/month, averaged across all NYC ZIP codes.
- **OMI** (Osservatorio del Mercato Immobiliare, Agenzia delle Entrate) for Rome, Milan, and Florence — semi-annual quotations of `loc_medio` (€/m²/month) for *abitazioni civili* (residential dwellings, `Cod_Tip = 20`) at the zona level, linearly interpolated to produce monthly estimates.

**Airbnb market pressure** is constructed from Inside Airbnb historical snapshots. For each city and each time period, the number of active *Entire home/apt* listings is computed per neighbourhood. Private rooms and shared rooms are excluded because only entire-unit withdrawals materially reduce the supply available to long-term tenants. Between snapshot dates, listing counts are held constant via forward-fill to produce a continuous monthly series.

**Key outputs:**
- `data/processed/<city>.csv` — all Airbnb snapshots merged per city (all room types)
- `data/processed/master_airbnb.csv` — all cities combined, filtered on Entire home/apt only
- `data/processed/omi_<city>.csv` / `omi_master.csv` — OMI residential quotations
- `data/processed/panel_italy.csv` — monthly panel: OMI rent (€/m²/mese) + n\_listings, Italian cities
- `data/processed/panel_ny.csv` — monthly panel: Zillow ZORI ($/mese) + n\_listings, New York

> ⚠️ **The two rent series are not comparable**: OMI measures €/m²/month (unit price per square metre), while ZORI measures absolute monthly rent in $. They operate on completely different scales and must never be combined in the same regression.

---

### Phase 2 — Exploratory Data Analysis (EDA)

Before any modelling, a systematic EDA characterises the data along six dimensions:

1. **Listing trends over time** — monthly active Entire home/apt listings for all four cities on a common timeline, with real snapshot dates marked as dots. The sharp contraction in New York following Local Law 18 is immediately visible as a near-vertical drop in September 2023.

2. **New York Local Law 18 inspection** — dual-axis chart overlaying listing volume (left axis, orange) and Zillow ZORI rent index (right axis, dark) with the September 2023 enforcement date annotated in red. This chart motivates the ITS modelling strategy by placing the supply shock and the rent series on the same timeline.

3. **OMI rent trends** — city-level `loc_medio` (€/m²/mese) for Rome, Milan, and Florence across the five available semesters (2023/1–2025/1), shown as connected dots at the real OMI reference dates (end of each semester).

4. **Airbnb price distributions** — box plots of nightly prices clipped at the 95th percentile, shown in two separate panels: Italian cities (EUR) and New York (USD), since the two currencies are not directly comparable. Each box shows the interquartile range of prices, with the white line indicating the median.

5. **Neighbourhood concentration** — horizontal bar charts of the top-10 most listed neighbourhoods per city based on the most recent available snapshot, revealing the geographic concentration of Airbnb supply within each city.

6. **New York seasonality heatmap** — a year × month grid of listing counts (only real snapshot months are filled; grey cells = no snapshot available). The LL18 shock in September 2023 is highlighted with a black rectangle, making seasonal patterns and the regulatory shock visible simultaneously.

---

### Phase 3 — New York: Interrupted Time Series (ITS)

New York provides the cleanest identification opportunity in the dataset. Local Law 18, enforced from September 2023, imposed a host registration requirement that caused active Entire home/apt listings to fall from approximately 24,000 to 11,500 within two months — a ~55% contraction. This constitutes a plausibly exogenous shock to short-term rental supply that is exploited as a natural experiment.

Two nested model specifications are estimated on **N = 34 monthly observations** (March 2023 – December 2025):

**BASE model:**

$$\text{ZORI}_t = \alpha + \beta_1 \, t + \beta_2 \cdot \mathbf{1}[t > T] + \beta_3 \, (t - T) \cdot \mathbf{1}[t > T] + \varepsilon_t$$

**EXTENDED model** (adds macroeconomic control and seasonal dummies):

$$\text{ZORI}_t = \alpha + \beta_1 \, t + \beta_2 \cdot \mathbf{1}[t > T] + \beta_3 \, (t - T) \cdot \mathbf{1}[t > T] + \beta_4 \cdot \text{FedRate}_t + \sum_{m=2}^{12} \gamma_m D_m + \varepsilon_t$$

The Fed funds rate is included to verify whether the ZORI dynamics are attributable to LL18 or instead to macroeconomic conditions. Monthly dummies absorb systematic seasonal patterns in rents.

**Variable legend:**

| Symbol | Type | Description |
|--------|------|-------------|
| $\text{ZORI}_t$ | **Target (dependent variable)** | Zillow Observed Rent Index at month $t$ — average monthly rent in $/month across all NYC ZIP codes. This is what the model tries to explain. |
| $t$ | Regressor | Time index: integer counting months elapsed from the start of the series (March 2023 = 0, April 2023 = 1, …, December 2025 = 33). Captures the baseline linear trend in rents before and after the intervention. |
| $T$ | Fixed constant | The intervention date: September 2023, encoded as the integer value of $t$ at that month. Everything after $T$ is "post-intervention". |
| $\mathbf{1}[t > T]$ | Regressor | Dummy variable equal to 1 for all months strictly after September 2023, 0 before. Captures any **immediate level shift** in rents at the moment of the intervention. |
| $(t - T) \cdot \mathbf{1}[t > T]$ | Regressor | Interaction term: months elapsed *since* the intervention, equal to 0 before $T$ and counting up from 1 afterward. Captures any **change in the slope** (trend) of rent growth after the intervention. |
| $\alpha$ | Parameter | Intercept — baseline rent level at $t = 0$ (March 2023). |
| $\beta_1$ | Parameter | **Pre-intervention trend** — monthly change in ZORI before September 2023, in $/month per month. |
| $\beta_2$ | Parameter | **Immediate level shift** — the jump (positive) or drop (negative) in ZORI on the month of intervention, in $/month. |
| $\beta_3$ | Parameter | **Post-intervention trend change** — the additional monthly slope after LL18 relative to the pre-intervention trend. |
| $\beta_4$ | Parameter | **Fed funds rate coefficient** (extended model only) — $/month change in ZORI per 1 pp increase in the federal funds rate. |
| $\gamma_m$ | Parameters | Monthly seasonal dummies (February–December; January = reference month). |
| $\varepsilon_t$ | Error | OLS residual. All inference uses HAC standard errors (Newey-West, maxlags = 4). |

**Estimation results (HAC standard errors, Newey-West maxlags = 4):**

| Parameter | BASE | ESTESO | Interpretation |
|-----------|:----:|:------:|----------------|
| β₁ — pre-LL18 trend | +43.21 $/mo *** | −6.44 $/mo ns | Trend absorbed by Fed rate in extended model |
| β₂ — level shift (Sep 2023) | **−78.88 $** * | **−88.14 $** ns | Immediate drop at LL18 enforcement — stable across specifications |
| β₃ — slope change | **−18.92 $/mo** *** | **+44.23 $/mo** ** | Sign reversal due to collinearity with Fed rate |
| β₄ — Fed rate | — | +208.17 $ * | Higher rates → reduced mortgage access → higher rents |
| R² | 0.829 | **0.930** | |
| Adjusted R² | 0.812 | **0.871** | |
| AIC | 421.9 | **415.6** | |

The stability of β₂ ≈ −79/−88 $ across both specifications is the most robust finding: LL18 caused an immediate ~$80/month drop in average NYC rents. The sign reversal in β₃ (from −19 to +44 $/month) reflects multicollinearity between the time trend and the Fed rate cycle; the extended model attributes the pre-LL18 growth mainly to rising rates rather than underlying trend.

**Cumulative effect at key horizons (extended model, delta method):**

| Horizon | Estimated effect | SE | Sig |
|---------|:---------------:|:--:|:---:|
| t+1 (1 month) | −44 $ | 84 | ns |
| t+6 (6 months) | +177 $ | 131 | ns |
| t+12 (12 months) | +443 $ | 234 | * |
| t+24 (24 months) | +973 $ | 462 | ** |

---

### Phase 3b — Out-of-Sample Forecast (March 2026 – March 2027)

Both ITS models are projected out-of-sample from the current date (March 2026) through March 2027. The extended model holds the Fed funds rate fixed at the last observed value in the panel; monthly seasonal dummies are constructed for each future month.

**Forecast assumptions:** (i) the post-LL18 trend continues at the estimated slope β₃; (ii) no new regulatory or macroeconomic shocks; (iii) the Fed rate remains at its last observed level (extended model only).

Key forecast horizons (point estimates, 95% CI covers parameter uncertainty only):

| Horizon | Date | BASE ($/mo) | EXTENDED ($/mo) |
|---------|------|:-----------:|:---------------:|
| +6 months | Sep 2026 | — | — |
| +12 months | Mar 2027 | — | — |

*Run `02_its_new_york.ipynb` cell 10 to obtain the numerical forecasts, which depend on the estimated parameters.*

> **Methodological caveat:** linear ITS forecasts become unreliable at long horizons. The 95% CIs widen substantially beyond the estimation sample. Results should be interpreted as scenario projections conditional on model assumptions, not as unconditional point predictions.

---

### Phase 4 — Italy: Cross-Sectional OLS per OMI zona

For the Italian cities, the time dimension is too thin (only 5 semesters) to support a credible time-series model. Instead, the analysis exploits **between-zone variation** within each city and across the three cities simultaneously.

The unit of observation is an **OMI zona × semester** cell. Each zona is matched to the count of active Airbnb listings in the corresponding neighbourhood via a spatial join between the Inside Airbnb `neighbourhood_cleansed` field and the OMI `LinkZona` identifier, using the GeoJSON boundary files in `geo/`. Listing density is computed as listings per km² (or per resident population) of the zona.

The model:

$$\text{loc\_medio}_{z,t} = \alpha + \beta \, \log(1 + \text{density}_{z,t}) + \gamma_f + \delta_c + \varepsilon_{z,t}$$

**Variable legend:**

| Symbol | Type | Description |
|--------|------|-------------|
| $\text{loc\_medio}_{z,t}$ | **Target (dependent variable)** | OMI average rental price in **€/m²/month** for zona $z$ at semester $t$. Computed as the midpoint of OMI's `Loc_min` and `Loc_max` quotations for *abitazioni civili* (`Cod_Tip = 20`). This is the price a landlord can expect per square metre of residential space in that zona. |
| $z$ | Index | OMI zona identifier (`LinkZona`) — the finest spatial unit in the OMI dataset, a sub-neighbourhood micro-zone. |
| $t$ | Index | Semester index: one of the 5 available periods (2023/1, 2023/2, 2024/1, 2024/2, 2025/1). |
| $\log(1 + \text{density}_{z,t})$ | Regressor | Log-transformed Airbnb listing density in zona $z$ at semester $t$. The $+1$ ensures the log is defined when density = 0 (zones with no Airbnb presence). Using log rather than levels compresses the right tail and gives $\beta$ a percentage-elasticity interpretation. |
| $\text{density}_{z,t}$ | Input to regressor | Number of active Entire home/apt Airbnb listings in the neighbourhood corresponding to zona $z$, divided by the zona's area in km² (or by resident population). |
| $\gamma_f$ | Fixed effect | **Fascia fixed effect** — a categorical dummy for each OMI fascia (A = centro storico, B = semicentro, C = periferia, D = suburbana, E = extraurbana). Absorbs the structural rent premium of central vs. peripheral locations, so that $\beta$ is not contaminated by the fact that both Airbnb and rents are higher in city centres. |
| $\delta_c$ | Fixed effect | **City fixed effect** — a dummy for each city (Roma, Milano, Firenze). Absorbs time-invariant differences in the overall rent level across cities (e.g., Milan's higher average rents vs. Florence). |
| $\alpha$ | Parameter | Intercept — baseline rent for a zona in the reference fascia and reference city with zero Airbnb density. |
| $\beta$ | Parameter | **Key coefficient of interest** — the semi-elasticity of rental prices to Airbnb density. Interpreted as: a 1% increase in $(1 + \text{density})$ is associated with a change of $\beta / 100$ €/m²/month in local rents, after controlling for city and fascia. |
| $\varepsilon_{z,t}$ | Error | OLS residual. Standard errors clustered at the zona level. |

**Cross-city comparison:** The main hypothesis is $\beta > 0$ — higher Airbnb density is associated with higher local rents, consistent with the supply-withdrawal mechanism. A secondary test checks whether $\beta$ is larger for Florence than for Rome or Milan. Florence has the highest Airbnb saturation in relative terms (listings per resident), so the supply-withdrawal effect should be most pronounced there if the channel is real.

---

### Phase 5 — Conclusions and Policy Recommendations

The final phase synthesises the empirical results from both the ITS model (New York) and the cross-sectional OLS (Italian cities) into a unified interpretation of the Airbnb effect on residential rental markets. We assess the strength and consistency of the evidence across cities and modelling strategies, discuss the limitations of the analyses, and reflect on the broader implications for platform economics.

On the basis of the findings, we formulate a set of **policy recommendations** targeted at urban regulators and local authorities. These include proposals on listing caps, host registration requirements, short-term rental taxation, and zoning instruments — drawing on the lessons from Local Law 18 in New York as a natural policy benchmark. The cross-city comparison allows us to calibrate recommendations to different market contexts, distinguishing between high-saturation tourist cities (Florence) and larger, more diversified rental markets (Milan, Rome, New York).

---

## Data Sources

### Airbnb listing data

| City | 2024 source | 2025 source | Total snapshots |
|------|-------------|-------------|-----------------|
| Rome | `airbnb.csv` — 4 quarterly snapshots (Mar, Jun, Sep, Dec 2024) | Inside Airbnb — 4 snapshots (Mar, Jun, Jul, Sep 2025) | 8 |
| Milan | `airbnb.csv` — 4 quarterly snapshots (Mar, Jun, Sep, Dec 2024) | Inside Airbnb — 2 snapshots (Jun, Sep 2025) | 6 |
| Florence | `airbnb.csv` — 4 quarterly snapshots (Mar, Jun, Sep, Dec 2024) | Inside Airbnb — 3 snapshots (Mar, Jun, Sep 2025) | 7 |
| New York | Kaggle archives — 2 snapshots (Mar 2023, Jan 2024) | Inside Airbnb — 9 snapshots (Mar–Sep, Nov, Dec 2025) | 11 |

**Room type normalisation:** The 2024 Italy source uses the label `"Entire home"` while Inside Airbnb 2025 uses `"Entire home/apt"`. Both are mapped to `"Entire home/apt"` in the master dataset. Only this category is retained for modelling.

### Residential price data

| City | Source | Variable | Unit | Frequency |
|------|--------|----------|------|-----------|
| Rome | OMI — Agenzia delle Entrate | `loc_medio` = (`Loc_min` + `Loc_max`) / 2, `Cod_Tip = 20` | €/m²/month | Semiannual → linearly interpolated to monthly |
| Milan | OMI — Agenzia delle Entrate | same | €/m²/month | Semiannual → linearly interpolated to monthly |
| Florence | OMI — Agenzia delle Entrate | same | €/m²/month | Semiannual → linearly interpolated to monthly |
| New York | Zillow ZORI (`new_york_data.csv`) | Mean ZORI across all NYC ZIP codes in Metro | $/month | Monthly (no interpolation needed) |

**OMI data structure:** Each semester provides two files per city — `VALORI` (price quotations per zona and property type) and `ZONE` (zona descriptions: name, fascia, microzona). The join key is `LinkZona`. Only `Cod_Tip = 20` (abitazioni civili, i.e. standard residential dwellings) is retained.

### Boundary files

GeoJSON neighbourhood boundary files are available for all four cities and will be used for spatial joins and choropleth visualisation.

---

## Repository Structure

```
data/
  airbnb/
    italy_2024/
      airbnb.csv                      Italy (Rome, Milan, Florence) — 4 quarterly snapshots 2024
    new_york_2023_2024/
      NYairbnb2023.zip                New York — single snapshot ~March 2023 (Kaggle)
      NYairbnb2024.zip                New York — single snapshot ~January 2024 (Kaggle)
      new_york_data.csv               Zillow ZORI — wide-format monthly time series by ZIP code
    insideairbnb_2025/
      Roma/                           Inside Airbnb .csv.gz snapshots (2025)
      Milano/                         Inside Airbnb .csv.gz snapshots (2025)
      Firenze/                        Inside Airbnb .csv.gz snapshots (2025)
      NewYork/                        Inside Airbnb .csv.gz snapshots (2025)
  omi/
    Roma/                             VALORI + ZONE CSVs: 2023/1, 2023/2, 2024/1, 2024/2, 2025/1
    Milano/                           VALORI + ZONE CSVs: 2023/1, 2023/2, 2024/1, 2024/2, 2025/1
    Firenze/                          VALORI + ZONE CSVs: 2023/1, 2023/2, 2024/1, 2024/2, 2025/1
  processed/
    roma.csv                          All Rome Airbnb snapshots merged (2024–2025), all room types
    milano.csv                        All Milan Airbnb snapshots merged (2024–2025), all room types
    firenze.csv                       All Florence Airbnb snapshots merged (2024–2025), all room types
    new_york.csv                      All New York Airbnb snapshots merged (2023–2025), all room types
    master_airbnb.csv                 All cities combined — Entire home/apt only (573,981 rows)
    omi_roma.csv                      OMI abitazioni civili (Cod_Tip=20) — Roma, all semesters
    omi_milano.csv                    OMI abitazioni civili — Milano, all semesters
    omi_firenze.csv                   OMI abitazioni civili — Firenze, all semesters
    omi_master.csv                    OMI abitazioni civili — all Italian cities combined
    panel_italy.csv                   Monthly panel: OMI loc_medio (€/m²/mese) + n_listings — Italian cities — OLS input
    panel_ny.csv                      Monthly panel: Zillow ZORI ($/mese) + n_listings — New York — ITS input
geo/
  neighbourhoodsMilano.geojson
  nieghborhoddsRoma.geojson
  neighbourhoodsNY.geojson
  airbnb_italy_city_neighbourhoods_geojson.geojson
plots/
  01_listing_trends.png               Monthly Entire home/apt listing trends for all cities (2023–2025)
  02_ny_ll18_zori.png                 NYC dual-axis: listing volume + Zillow ZORI, with LL18 annotation
  03_omi_trends.png                   OMI loc_medio (€/m²/mese) for Italian cities — real semester dots
  04_price_distribution.png           Airbnb nightly price distribution per city (boxplot, clipped at 95th pct)
  05_top_neighbourhoods.png           Top-10 neighbourhoods by listing count — most recent snapshot per city
  06_ny_heatmap.png                   NYC listing counts: year × month heatmap (real snapshots only)
01_eda_airbnb.ipynb                   Main notebook — Phase 1 preprocessing + Phase 2 EDA (Steps 1–3)
02_its_new_york.ipynb                 ITS analysis — Phase 3: BASE + ESTESO models, diagnostics, cumulative effects, out-of-sample forecast (Mar 2026 – Mar 2027)
```
