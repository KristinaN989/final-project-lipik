# Agricultural Drought Prediction for Eastern Croatia

AI/ML student project — predicting agricultural drought in Slavonia two months ahead using Copernicus satellite data.

---

## How to Run

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Run `prepare_dataset.ipynb` first — loads raw data, computes labels and features, saves `data/dataset.csv`

3. Run `drought_prediction.ipynb` — trains models, evaluates, saves best model

> Raw data files (`era5_land.nc`, `satellite_sm.nc`) need to be downloaded separately — see below.

---

## Datasets

### ERA5-Land Monthly Means
- **Source:** Copernicus Climate Change Service
- **Link:** https://cds.climate.copernicus.eu/datasets/reanalysis-era5-land-monthly-means
- **Variables:** 2m temperature, dewpoint, volumetric soil water layers 1–4, precipitation, evaporation, solar radiation
- **Period:** 2003–2024 | **Resolution:** 0.1° | **Region:** lat 45.0–45.8°N, lon 17.5–19.0°E

### ESA CCI Satellite Soil Moisture
- **Source:** Copernicus Climate Change Service
- **Link:** https://cds.climate.copernicus.eu/datasets/satellite-soil-moisture
- **Variable:** Surface soil moisture volumetric (combined passive+active microwave, CDR v202505)
- **Period:** 2003–2024 | **Resolution:** 0.25° (native sensor resolution)

Both datasets were downloaded via the CDS API. The satellite SM (0.25°) was regridded to match ERA5 (0.1°) using nearest-neighbour interpolation, which is standard practice since no higher native resolution exists for this product.

**Final dataset:** 37,728 observations, 26 columns.

---

## Drought Labels

No pre-labelled dataset was used — drought labels were computed from the data using the **SSI-3** (Standardised Soil-moisture Index, 3-month) approach:

1. **Root-zone soil moisture** — ERA5 soil layers 1–3 combined with depth weights (swvl1×0.07 + swvl2×0.21 + swvl3×0.72), representing the 0–100 cm agricultural root zone
2. **3-month smoothing** — rolling mean per grid cell to capture sustained deficits rather than single dry months
3. **Standardised anomaly** — z-score against the long-term monthly mean and std per grid cell
4. **Threshold:** anomaly < −0.5 → drought, otherwise normal
5. **2-month lead** — label shifted 2 months forward so the model forecasts future drought from current conditions

**Sources used for methodology:**
- McKee, T.B. et al. (1993). *The relationship of drought frequency and duration to time scales.* 8th Conference on Applied Climatology — original SPI drought classification thresholds (−0.5, −1.0) adopted for SSI
- Hao, Z. & AghaKouchak, A. (2013). *Multivariate Standardized Drought Index.* Advances in Water Resources. https://doi.org/10.1016/j.advwatres.2013.03.009 — SSI z-score methodology
- ECMWF ERA5-Land documentation — soil layer depths. https://confluence.ecmwf.int/display/CKB/ERA5-Land
- Cammalleri, C. et al. (2021). *A revision of the Combined Drought Indicator (CDI) used in the European Drought Observatory.* NHESS. https://doi.org/10.5194/nhess-21-481-2021 — reference for soil-moisture-based drought forecasting lead time

---

## Features

| Feature | Description |
|---|---|
| `t2m`, `d2m` | 2m temperature and dewpoint (K) |
| `swvl1–4` | Volumetric soil water, layers 1–4 (m³/m³) |
| `tp` | Total precipitation (m) |
| `pev`, `e` | Potential and actual evaporation (m) |
| `ssr` | Surface net solar radiation (J/m²) |
| `sat_sm` | Satellite surface soil moisture (m³/m³) |
| `sm_rootzone` | Depth-weighted root-zone moisture (m³/m³) |
| `sm_anomaly` | SSI-3 anomaly (z-score) |
| `month` | Calendar month |
| `swvl1_lag1/2/3` | Soil moisture from 1, 2, 3 months prior |
| `tp_roll3`, `t2m_roll3` | 3-month rolling means |
| `tp_deficit` | Precipitation minus long-term monthly mean |

---

## Requirements

See `requirements.txt`.
