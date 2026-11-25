# Reconstructing Aircraft Traffic Patterns from OpenSky Data

Final project for **Fundamentals of Digital Archeology** (University of Tennessee, Knoxville).
Team: Marwan Elawady · Gasser Elkhodary · Amira Alkafoury

We collect aircraft state observations from the OpenSky Network around three U.S. hubs (ATL, ORD, DFW; 80 km radius), clean them, reconstruct candidate track segments, and compare the regions. Every number in the final report comes from saved run **`20261110T023641574097Z`**, which is included in this repository.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/GasserElkhodary/aircraft-traffic-reconstruction/blob/main/Aircraft_Traffic_Final.ipynb)

## Repository layout

```
Aircraft_Traffic_Final.ipynb      Complete pipeline: collection, cleaning, reconstruction, analysis, validation, figures
requirements.txt                  Python dependencies
live_data/
  states_2026-11-10-02.csv        Raw input: 25,091 OpenSky state observations (Nov 10, 2026, 02:37:40–02:52:34 UTC)
  SHA256SUMS                      Checksum of the raw input (matches source_manifest.json)
aircraft_final/
  data/regional/                  Regional partitions (Parquet) produced from the raw input
  metadata/latest_run.json        Pointer to the saved run
  reports/latest_report.md        Auto-generated run report
  results/20261110T023641574097Z/ Saved run used in the report (tables, tracks, figures, provenance)
docs/
  Aircraft_Traffic_Final_Report.docx / .pdf   Final report
  Aircraft_Traffic_Final_Presentation.pptx     Final presentation
```

## Data source

- **OpenSky Network live REST API**, endpoint `/api/states/all` ([docs](https://openskynetwork.github.io/opensky-api/rest.html)), polled with one bounding box per airport about every 10 seconds.
- **Scope:** one 14 min 54 s window on 2026-11-10 (UTC). Each row is one aircraft state at one timestamp, not one flight.
- **Why not the planned historical data:** the original plan used four Mondays in April 2022 from OpenSky's historical state-vector archive. Archive requests returned `NoSuchKey`, so we switched to live collection. The notebook still supports archive input (`DATA_MODE='archive'`) if the archive becomes available again.
- **Integrity:** SHA-256 of the raw CSV is `9253df4f00cd87ba21b3082028189a595ba5f4dda3423023f913d4601dcc181d`. Check it with `sha256sum -c live_data/SHA256SUMS`.

## Reproduce the reported results

The included CSV lets you reproduce the saved run without contacting OpenSky.

**Locally**
```bash
git clone https://github.com/GasserElkhodary/aircraft-traffic-reconstruction.git
cd aircraft-traffic-reconstruction
pip install -r requirements.txt
jupyter notebook Aircraft_Traffic_Final.ipynb   # then Run All
```

**Google Colab:** open the notebook using the badge above, upload `live_data/states_2026-11-10-02.csv` to `/content/live_data/`, then Run All.

The notebook reads every `*states_*.csv` file in `live_data/` (`DATA_MODE='live'`, `COLLECT_MINUTES=0`). It writes a new run folder under `aircraft_final/results/<new run id>/`. The CSV outputs match the saved run (verified by re-executing the notebook; only row order and the last floating-point digits differ).

The original run used Python 3.14, pandas 3.0.5, numpy 2.4.6, pyarrow 26.0.1 and matplotlib 3.11.1 (see `results/20261110T023641574097Z/environment.json`). The re-execution check used Python 3.13, pandas 3.0.5, numpy 2.5.3, pyarrow 26.0.1 and matplotlib 3.11.2.

### Collecting new data (optional)

Set `COLLECT_MINUTES` to a positive number (for example, 15) and run on a laptop. Cloud IPs, including Colab, are often blocked by OpenSky. Anonymous access works within OpenSky's daily quota. Optional OAuth client credentials can be passed to `collect_live(...)`; never commit them. New data gives different results because traffic, receiver coverage and API availability change.

## Key parameters

Set in the configuration cell of the notebook and recorded in `results/<run>/config.json`:

| Parameter | Value | Meaning |
|---|---|---|
| `RADIUS_KM` | 80 | Great-circle radius around each airport reference point |
| `STALE_SECONDS` | 60 | Drop rows whose contact/position age is > 60 s or negative |
| `GAP_SECONDS` | 300 | Start a new track segment after a gap > 300 s |
| `SPEED_FLAG_MPS` | 350 | Flag reported or implied speeds above this |

Sensitivity runs: freshness 15/60/120 s, gap 120/300/600 s, speed 250/350/450 m/s.

## Where each report result comes from

All files are in `aircraft_final/results/20261110T023641574097Z/`.

| Report item | File |
|---|---|
| Table 3 (preprocessing stages), per-region stale rows | `quality.csv`, `source_manifest.json` |
| Table 4 and missingness text | `missingness.csv` |
| Table 5 (regional comparison) | `region_profile.csv`, `daily.csv` |
| Segment counts and durations (§5.2) | `region_totals.csv`, `segments.csv`, `gaps.csv` |
| Flagged observations (§5.2) | `flags.csv` |
| Table 6 and Figure 3 (threshold sensitivity) | `threshold_sensitivity_total.csv`, `sensitivity.csv`, `06_sensitivity.png` |
| Freshness ranking and speed-threshold results (§5.3) | `freshness_region_counts.csv`, `speed_sensitivity.csv` |
| Figure 1 (sample tracks) | `07_sample_track_review.png`, `tracks_*.parquet` |
| Figure 2 (spatial density) and densest cells | `04_spatial_density.png`, `spatial_bins.csv`, `high_observation_bins.csv` |
| 18 integrity checks and 9 lineage audits | `validation.csv`, `audits.csv` |
| Run configuration, environment and provenance | `config.json`, `environment.json`, `source_manifest.json`, `status.json` |

Paths inside the saved JSON files (for example `C:\Users\...`) are the original machine's paths. They are kept for provenance and are not needed to rerun the notebook.

## Limitations

The sample is about 15 minutes from one night, so it supports no daily, weekly or seasonal conclusions. OpenSky receiver coverage is uneven and unknown. Roughly 38% of rows lack altitude or vertical rate. Track segments are candidate observed movements, not complete flights, and are not verified against flight schedules. See Section 6 of the report.

## Team contributions

- **Marwan Elawady:** data acquisition, source provenance, integration
- **Amira Alkafoury:** cleaning, validation, track reconstruction
- **Gasser Elkhodary:** regional analysis, visualizations, sensitivity experiments, interpretation
