# Reconstructing Aircraft Traffic Patterns from OpenSky Data
Marwan Elawady · Gasser Elkhodary · Amira Alkafoury

## Study status
**LIVE-DATA RUN — VALIDATED (1 day(s), 1 UTC hour(s) with data)**
Data mode: live. Dates (UTC): 2026-11-10. Regions: ATL, ORD, DFW; radius 80 km.
Source: OpenSky live REST API polled every 10 s around each airport; 1 UTC hour(s) with data. The planned 2022 archive (four Mondays) was emptied by OpenSky on 2026-11-04, so this run demonstrates the pipeline on current data from the same network and schema.
Source rows scanned: 25,091; regional rows before cleaning:
23,466; retained rows: 19,604.

## Research question
How do observed aircraft activity, spatial concentration, and movement characteristics vary
between three airport regions and four weekly snapshot days? Which unusual observations
or reporting patterns merit investigation?

## Methods
Read hourly CSV files (live collection) or CSV/gzipped CSV records directly from hourly TAR archives; retain source-file checksums
and row lineage. Standardize Unix seconds to UTC, reject invalid coordinates, and select points
within an approximate 80 km great-circle radius of each airport reference point. Deduplicate
region/aircraft/timestamp keys, keeping the most recent contact timestamp; report conflicting
positions. Retain missing optional measurements; remove known freshness ages above 60 seconds
or below zero. Segment each aircraft's within-region, within-day observations at gaps over
300 seconds; do not split at hourly file boundaries. Aggregate observation counts, distinct
aircraft per hour, movement medians, and approximate 2 km spatial bins. Compare matched acquired
hours; distinguish unknown hours from zero retained observations. Inspect high-speed and position
jump candidates and leave-one-day-out hourly count ratios (only when several days are available).

## Results and conclusions
- Across 1 UTC hour(s) represented on every selected day and in every region, DFW had the highest mean retained observation count per acquired hour (7,537). This is an observation-based comparison, not airport throughput.
- Region profile (retained rows / distinct aircraft-days / median ground speed / median baro altitude): ATL: 6,366 / 126 / 58 m/s / 2,614 m; ORD: 5,701 / 153 / 94 m/s / 1,920 m; DFW: 7,537 / 151 / 87 m/s / 1,455 m. DFW had the most retained observations and ORD the most distinct aircraft in the available data.
- Cleaning retained 19,604 of 23,466 regional rows (83.5%). 0 duplicates and 3,862 stale/future-timestamp rows were excluded.
- 440 candidate within-region segments were reconstructed; 437 contain multiple observations. They do not identify complete flights or scheduled routes.
- 3 observations triggered at least one movement/data-quality flag. 0 hourly count deviations met the exploratory ratio rule. Neither establishes a confirmed disruption.
- The available-data observation-count leader remained DFW across freshness thresholds of 15, 60, and 120 seconds. This sensitivity result does not correct receiver coverage bias.

## Validation
Row accounting, duplicate keys, track ordering, gap limits, radial bounds, and segment totals
are checked for every processed region-day. Representative short, medium, and long segments
are linked to imported source rows. This is an automated lineage audit; a human track review
is not claimed. Sensitivity uses freshness thresholds 15/60/120 seconds, track gaps 120/300/600
seconds, and speed flag thresholds 250/350/450 m/s. See the exported validation and sensitivity tables.

## Limitations
Observation counts measure retained reports, not departures, arrivals, flight counts, or airport
capacity. Receiver coverage, sampling, stale states, missingness, and filtering influence the
results. The collected days do not describe other weekdays or seasons. Region boundaries truncate
trajectories; identifiers can recur across flights. Local-clock comparisons use selected UTC
days, including portions of adjacent local dates. Spatial bins are approximate. Candidate flags
are exploratory and are not confirmed incidents. Three reference Mondays are insufficient for
strong inferential claims about unusual traffic. No official schedule/status dataset was matched,
so cancellations and official delays are outside the evidence. The 2022 source archives were removed
by OpenSky on 2026-11-04, which prevents completion of the planned scope; no data or findings are fabricated to fill them.

## Reproducibility
The notebook writes source_manifest.json, coverage.csv, quality.csv, validation.csv, audits.csv,
threshold and speed sensitivity tables, per-region-day Parquet tracks, figures, and this report.
Review source terms before sharing raw records. Raw source data is fetched or supplied locally,
not embedded in the notebook.

## References
- The OpenSky Network, https://opensky-network.org
- Scientific datasets: https://opensky-network.org/data/scientific
- State-vector schema/units: https://openskynetwork.github.io/opensky-api/trino.html
- Schäfer et al. (2014), Bringing up OpenSky: A large-scale ADS-B sensor network for research.
- Team project proposal, Transportation_Data_Analysis_Project_Proposal.pdf.
