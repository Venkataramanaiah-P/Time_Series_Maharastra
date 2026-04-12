# 📈 NDVI Time Series Analysis — Maharashtra
### Google Earth Engine Python API · Mann-Kendall Trend · Sentinel-2 / Landsat

[![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)](https://python.org)
[![GEE](https://img.shields.io/badge/Google%20Earth%20Engine-API-green)](https://earthengine.google.com)
[![Sentinel-2](https://img.shields.io/badge/Sentinel--2-SR%20Harmonized-orange)]()
[![Landsat](https://img.shields.io/badge/Landsat-8%20%2F%209-red)]()
[![License](https://img.shields.io/badge/License-MIT-lightgrey)](LICENSE)

---

## 📌 Project Overview

Multi-year **NDVI time series analysis** for Maharashtra state tracking seasonal vegetation dynamics and long-term greening/browning trends using Google Earth Engine Python API.

Monthly NDVI composites are extracted from Sentinel-2 or Landsat ImageCollections. A **Mann-Kendall trend test** is applied to detect statistically significant vegetation change, and pixel-wise slope maps show spatially explicit greening or browning patterns.

---

## 🗺️ Output Maps

![NDVI Time Series Chart](outputs/ndvi_timeseries_chart.png)

> Fig 1. Monthly NDVI Time Series — Maharashtra (2018–2023)

![Trend Slope Map](outputs/ndvi_slope_map.png)

> Fig 2. Pixel-wise NDVI Slope Map — Green = Greening · Red = Browning

---

## 📊 Key Results

| Metric | Value |
|---|---|
| Study Area | Maharashtra State (~307,713 km²) |
| Date Range | 2018 – 2023 (5 years) |
| Satellite | Sentinel-2 SR Harmonized |
| Temporal Resolution | Monthly median composites |
| Trend Test | Mann-Kendall (non-parametric) |
| Outputs | Time series chart + slope map + anomaly map |

### Mann-Kendall Trend Summary

| Region | Trend Direction | Sen's Slope | Significance |
|---|---|---|---|
| Western Ghats | Greening ↑ | +0.004/yr | p < 0.05 |
| Vidarbha (east) | Browning ↓ | −0.002/yr | p < 0.05 |
| Marathwada | No change | ±0.001/yr | p > 0.10 |
| Pune Metro | Browning ↓ | −0.003/yr | p < 0.05 |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| Google Earth Engine Python API | ImageCollection pipeline |
| `ee.ImageCollection` + median reducer | Monthly cloud-free composites |
| Mann-Kendall test (`pymannkendall`) | Non-parametric trend detection |
| `matplotlib` / `pandas` | Time series charting + CSV export |
| Sentinel-2 SR / Landsat 8-9 | Source imagery |
| `geemap` | Map export and visualization |

---

## 📁 Project Structure

```
Time_Series_Maharastra/
├── notebooks/
│   └── ndvi_timeseries.ipynb       ← Main Colab notebook
├── scripts/
│   ├── build_imagecollection.py    ← ImageCollection + cloud masking
│   ├── compute_timeseries.py       ← Monthly composites + CSV export
│   ├── mannkendall_trend.py        ← Trend test + slope map
│   └── export_maps.py              ← GeoTIFF + PNG export
├── outputs/
│   ├── ndvi_timeseries_chart.png   ← Monthly NDVI line chart
│   ├── ndvi_slope_map.png          ← Greening/browning map
│   ├── ndvi_anomaly_map.png        ← Deviation from long-term mean
│   ├── ndvi_timeseries.csv         ← Raw monthly NDVI values
│   └── trend_summary.csv           ← Mann-Kendall results per zone
├── requirements.txt
└── README.md
```

---

## 🚀 How to Run

```bash
pip install earthengine-api geemap pymannkendall pandas matplotlib
earthengine authenticate
python scripts/compute_timeseries.py
```

---

## 🔑 Core Code

```python
import ee, geemap
ee.Initialize()

maharashtra = ee.FeatureCollection('FAO/GAUL/2015/level1') \
               .filter(ee.Filter.eq('ADM1_NAME', 'Maharashtra')).geometry()

# Cloud mask
def mask_s2(img):
    qa = img.select('QA60')
    mask = qa.bitwiseAnd(1<<10).eq(0).And(qa.bitwiseAnd(1<<11).eq(0))
    return img.updateMask(mask).divide(10000)

# Build ImageCollection 2018-2023
collection = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
    .filterBounds(maharashtra)
    .filterDate('2018-01-01', '2023-12-31')
    .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 5))
    .map(mask_s2))

# Monthly NDVI composites
def monthly_ndvi(year, month):
    img = collection.filter(ee.Filter.calendarRange(year, year, 'year')) \
                    .filter(ee.Filter.calendarRange(month, month, 'month')) \
                    .median().clip(maharashtra)
    ndvi = img.normalizedDifference(['B8', 'B4']).rename('NDVI')
    return ndvi.set({'year': year, 'month': month,
                     'system:time_start': ee.Date.fromYMD(year, month, 1).millis()})

# Build monthly time series
years  = ee.List.sequence(2018, 2023)
months = ee.List.sequence(1, 12)
monthly = ee.ImageCollection(
    years.map(lambda y: months.map(lambda m: monthly_ndvi(y, m))).flatten()
)

# Export time series chart data to CSV
chart_data = monthly.getRegion(maharashtra, 10000)
```

---

## 📦 Deliverables

| File | Description |
|---|---|
| `ndvi_timeseries_chart.png` | Monthly NDVI line chart (PNG) |
| `ndvi_slope_map.tif` | Pixel-wise trend slope GeoTIFF |
| `ndvi_anomaly_map.tif` | Deviation from long-term mean |
| `ndvi_timeseries.csv` | Raw monthly NDVI values |
| `compute_timeseries.py` | Reusable GEE Python pipeline |
| `findings_report.pdf` | Seasonal patterns + key findings |

---

## 👤 Author

**Venkataramanaiah**
GIS Analyst · Google Earth Engine · PyQGIS · Python
📍 Ahmedabad, India
🔗 [Upwork Profile](https://www.upwork.com/freelancers/venkataramanaiah)
▶️ [Portfolio Video](https://youtu.be/6bctiAni4Ug)

---

## 📄 License

MIT License — free to use and adapt with attribution.
