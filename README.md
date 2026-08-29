# Structure, Imbalance and Fleet Behaviour in the London Cycle Hire Network

A visualisation-led investigation of 17 months of TfL Cycles journey data, covering 12.5 million hires across 797 docking stations between 1 January 2025 and 31 May 2026.

---

## Repository contents

| Path | Contents | Committed |
|---|---|---|
| `DSM050_CW2_BulentAlptekin.ipynb` | Full analysis, from data acquisition to figures | Yes |
| `DSM050_CW2_BulentAlptekin.html` | Executed export of the notebook, viewable without running it | Yes |
| `data/clean/journeys_clean_sample.csv` | 1,000-row illustrative sample of the analytical dataset | Yes |
| `data/reference/stations_bikepoint.csv` | Pinned BikePoint snapshot, retrieved 15 August 2026 | Yes |
| `data/raw/` | TfL journey extracts as downloaded | No, created on first run |
| `data/reference/` | Station crosswalk, cleaning and resolution ledgers, schema audit, elevation cache, data dictionary, basemap rasters | No, written on first run |
| `data/cache/` | Per-file Parquet cache of the raw extracts | No, created on first run |
| `data/clean/` | The full analytical dataset, 17 monthly Parquet partitions | No, written on first run |
| `figures/` | All 23 report figures as PNG, plus the Plotly HTML for Figure 21 | Yes |
| `requirements.txt` | Python dependencies | Yes |

---

## Data sources

**Journey records.** Transport for London Cycles open data, published as monthly CSV extracts in a public S3 bucket: <https://cycling.data.tfl.gov.uk>. The notebook derives the analytical window from four explicit criteria (schema homogeneity, unbroken daily coverage, a complete calendar year, complete final month) and then pins `WINDOW_END` to 31 May 2026 so that later publications by TfL do not change the result.

**Station reference.** TfL Unified API BikePoint endpoint: <https://api.tfl.gov.uk/BikePoint>. This endpoint returns the *live* network, so its contents change as stations are commissioned, renamed and decommissioned. The response retrieved on 15 August 2026 is supplied as `data/reference/stations_bikepoint.csv` and loaded from disk rather than refetched. Refetching would alter the station crosswalk and every station-level result downstream, including the clustering in RQ5. The retrieval code is retained in the notebook, unexecuted, for transparency.

**Supporting data.** UK bank holidays from <https://www.gov.uk/bank-holidays.json>, and station elevations from the Open-Meteo elevation API. Elevations are cached to `data/reference/station_elevation.csv` after first retrieval.

Basemap tiles are Esri WorldGrayCanvas, which requires no API key. They are fetched once on first run and cached to local GeoTIFF rasters, after which the map figures make no further tile requests. The rasters are not committed here on size grounds.

### Sample of the analytical dataset

The full cleaned dataset runs to 12.5 million rows across 17 monthly Parquet partitions and is too large to commit. It is written to `data/clean/` when the notebook is run, and regenerates exactly, because both the analytical window and the station reference are pinned.

`data/clean/journeys_clean_sample.csv` is committed in its place: a 1,000-row sample drawn across the full window with a fixed seed, carrying all 28 variables including the derived ones (`period`, `is_commuter_trip`, `straight_km`, `duration_min`, `season`, `is_round_trip`). It is there so the structure and typing of the data can be inspected without cloning or running anything, and opens directly in the GitHub file view. It is illustrative only; no result in the report can be reproduced from it.

---

## Running the notebook

```bash
pip install -r requirements.txt
jupyter lab
```

Python 3.14.0.

The first run downloads roughly 2 GB of CSV extracts from TfL and takes some time. It also fetches the basemap rasters and the station elevations, both of which are then cached. Subsequent runs read from the Parquet cache in `data/cache/` and are much faster. Beyond those first-run fetches and the bank holiday lookup, the notebook reads only from files committed to this repository.

Run the notebook top to bottom. Cells are ordered and several depend on state established earlier.

---

## Figure index

| Fig. | File | Report section |
|---|---|---|
| 1 | `fig01_schema_eras.png` | 3.1 |
| 2 | `fig02_cleaning_ledger.png` | 3.2 |
| 3 | `fig03_station_concentration.png` | 3.3 |
| 4 | `fig04_commuter_hours.png` | 4.1 |
| 5 | `fig05_duration_distribution.png` | 4.1 |
| 6 | `fig06_duration_quantiles.png` | 4.1 |
| 7 | `fig07_daily_demand.png` | 4.1 |
| 8 | `fig08_hour_weekday_heatmap.png` | 4.1 |
| 9 | `fig09_autocorrelation.png` | 4.1 |
| 10 | `fig10_stl_decomposition.png` | 4.1 |
| 11 | `fig11_seasonal_yoy.png` | 4.1 |
| 12 | `fig12_station_commute_signature.png` | 4.2 |
| 13 | `fig13_net_flow_map.png` | 4.2 |
| 14 | `fig14_network_structure.png` | 4.3 |
| 15 | `fig15_waterloo_ego.png` | 4.3 |
| 16 | `fig16_hyde_park_ego.png` | 4.3 |
| 17 | `fig17_ebike_supply.png` | 4.4 |
| 18 | `fig18_distance_by_fleet.png` | 4.4 |
| 19 | `fig19_ebike_gap_map.png` | 4.4 |
| 20 | `fig20_k_selection.png` | 4.5 |
| 21 | `fig21_pca_3d_interactive.html` and `.png` | 4.5 |
| 22 | `fig22_cluster_map.png` | 4.5 |
| 23 | `fig23_radar_profiles.png` | 4.5 |

Figure 21 is a rotatable three-dimensional scatter. The PNG is a static capture for the report; open `fig21_pca_3d_interactive.html` for the interactive version.

---

## Reproducibility

Four sources of drift are pinned so that a later run reproduces the reported numbers:
- **Analytical window.** `WINDOW_END` is fixed at 31 May 2026 rather than derived from whatever files the S3 bucket holds at run time.
- **Station reference.** Loaded from a dated snapshot rather than the live BikePoint endpoint, for the reason given above.
- **Clustering.** `KMeans` is fitted with `random_state=42` and `n_init=20`. Cluster names are derived from centroid values rather than from the arbitrary integer labels k-means assigns.
- **Basemaps.** Tile rasters are fetched once and cached locally, so repeated runs of the map figures do not depend on tile service availability.
