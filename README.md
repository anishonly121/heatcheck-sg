# HeatCheck

**A day's warning of dangerous heat for Active Ageing Centres, by area, on WhatsApp.**

Built for the Databricks AI Social Impact (DAISI) Challenge, Singapore Edition 2026. Problem statement B1: HeatGuard, urban heat risk mapping for vulnerable communities.

## The problem

Singapore had at least 29 days of high heat stress in 2025, and it is now a super-aged society. 69,650 seniors aged 65 and above live in HDB 1-2 room flats, and 88,400 seniors live alone. Poverty, poor health and isolation all raise the risk of dying from heat.

Active Ageing Centres (AACs) are the ones who look out for these seniors, but they have no early warning for heat. When I interviewed AAC staff, they told me they don't check heat forecasts. Decisions depend on what staff see on the day, and they worry most about their frailest seniors, some in their 90s and 100s.

## What HeatCheck does

HeatCheck combines NEA heat stress data with where vulnerable seniors live, and ranks every residential subzone into one of three actions:

- **Check first:** the highest-risk 10% of subzones
- **Check next:** the next 20%
- **Monitor:** the rest

**Risk = heat x seniors in HDB 1-2 room flats x distance from eldercare**

The first version already runs on real data. 23 of 216 scored subzones are "Check first", and together they hold 34% of all seniors living in HDB 1-2 room flats. The top 66 subzones hold 80%. This means AACs can reach most of the seniors who need help the most by focusing on under a third of the island.

## How it works

Everything runs in Databricks Free Edition.

| Notebook | What it does |
|---|---|
| `00_setup_and_test` | Creates the Unity Catalog schema and volume, and checks that data.gov.sg is reachable |
| `01_ingest_live` | Saves live WBGT and air temperature readings as raw JSON files |
| `02_bronze_load` | Auto Loader picks up only new files and loads them into bronze Delta tables |
| `03_silver` | Parses the JSON into clean tables, with quality checks for missing and unrealistic values |
| `04_backfill_wbgt` | Pulls every WBGT reading since February 2025 using the API's date parameter |
| `05_reference_data` | Loads seniors by subzone (SingStat), subzone boundaries (URA) and eldercare and AAC locations (MOH) |
| `06_risk_v0` | Builds the gold risk table, a static map for slides and an interactive HTML map |
| `07_pitch_numbers` | Pulls the key numbers used in the pitch |
| `08_forecast_test` | Tests a next-day WBGT forecast against simple baselines, tracked in MLflow |

Notebooks 01 to 03 run as a Lakeflow Job every hour, so the data stays live without me touching it.

## Data (all open data)

- WBGT heat stress, live and history since Feb 2025 (NEA, data.gov.sg)
- Air temperature, live readings (NEA, data.gov.sg)
- Singapore Residents by Planning Area/Subzone, Age Group, Sex and Type of Dwelling, June 2026 (SingStat)
- Eldercare Services and Dementia Go-To-Points, filtered to Active Ageing Centres (MOH, data.gov.sg)
- Master Plan 2019 Subzone Boundary (URA, data.gov.sg)

## Results so far

- About 950,000 WBGT readings from 31 stations. Quality checks removed 0.3% that were missing or unrealistic.
- 216 subzones scored, 23 marked "Check first".
- **Forecast test:** my first next-day model did not beat simple baselines. Its average error was 1.01°C, compared with 0.97°C for "tomorrow = today" and 0.87°C for a 7-day average. Past heat alone isn't enough to predict tomorrow, so the next version adds NEA's own 24-hour forecast as an input.

## Limitations

- Heat for each subzone comes from the nearest of 31 stations, so it is an estimate.
- Seniors in HDB 1-2 room flats stand in for low income. It is a proxy, not a direct measure.
- The eldercare list has 163 locations, while there are over 200 AACs nationwide.
- The risk score has not yet been validated against real heat illness data, because that data is not public.

## What's next

- Add NEA's 24-hour forecast so the next-day model beats simple baselines
- Add seniors aged 85+ to the risk score, as AAC staff asked
- A daily WhatsApp-ready message for AAC staff, a Databricks App with the map, and Genie for plain-English questions
- Test it with the AAC staff I interviewed

## Running it yourself

1. Create a Databricks Free Edition workspace and clone this repo as a Git folder.
2. Get a data.gov.sg API key and store it as a secret: scope `heatcheck`, key `datagov_api_key`. Never put the key in a notebook.
3. Download the SingStat CSV above and upload it to `/Volumes/workspace/heatcheck/raw/singstat/`.
4. Run the notebooks in order from 00 to 08.

## Author

Anish, Year 3 Information Technology, Singapore Polytechnic.

Data from data.gov.sg and SingStat is used under the Singapore Open Data Licence. Thank you to the AAC staff who took time to share how they handle hot days.
