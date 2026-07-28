

# Data Center-Renewables Exploration


The project title should be concise and self-explanatory so that the user can easily remember your project.

Add a cover banner to the top of your Readme to catch the attention of your readers.
I usually include images that are relevant to my project, and you can easily find any image for free online without worrying about copyright issues. However, if the work is not free, make sure to credit the proper owners in the references/acknowledgement section.

The colorful tiles beneath the title are known as badges, and they improve readability by providing quick insights into the github repository. I use [Shields IO](https://shields.io/). Depending on the project you can use the ones that are relevant. 

# Project Overview

This project is about exploring the relationships between data centers power capacity/demand, commercial renewable energy production, total commercial energy production, retail electricity prices, and thermoelectric water withdrawals, all by state in The United States. It seeks to answer the following question: How does sample data center power demand vary across states in relation to renewable electricity generation share, electricity prices, and thermo-electric water withdrawals? 
Subquestions: Which states have the highest sampled data center power capacity?
Do states with larger sample data center power capacity tend to have lower or higher retail electricity prices?
Are states with larger electricity systems also the states with greater sample data center power capacity?

## Codes and Resources Used
In this section I give user the necessary information about the software requirements.
- **Editor Used:**  Visual Studios Code
- **Python Version:** Python 3.12.13 

## Python Packages Used
In this section, I include all the necessary dependencies needed to reproduce the project, so that the reader can install them before replicating the project. I categorize the long list of packages used as - 

- **General Purpose**: urllib3, requests, PySocks, certifi, charset-normalizer, idna, httpx, httpcore, h11, h2, hpack, hyperframe, anyio, sniffio, click, colorama, tqdm, packaging, setuptools, pip, filelock, platformdirs, typing_extensions, typing-inspection, importlib_metadata, zipp, six, python-dateutil, pydantic, pydantic_core, annotated-types, Jinja2, MarkupSafe, jsonpatch, jsonpointer, tomli, build, pyproject_hooks, installer, distro, exceptiongroup, cffi, pycparser, msgpack, ruamel.yaml, ruamel.yaml.clib, zstandard, backports.zstd, Brotli, truststore, unearth, boltons, cached-property, frozendict, conda, conda_index, conda-libmamba-solver, conda-lockfiles, conda-package-handling, conda_package_streaming, conda-pypi, conda-rattler-solver, conda-self, libmambapy, py-rattler, pycosat, archspec, menuinst 
- **Data Manipulation**: pandas, numpy, scipy, statsmodels, patsy
- **Data Visualization:** matplotlib, seaborn, pillow, fonttools, contourpy, cycler, kiwisolver, pyparsing, munkres

# Methodology

## Source Data

This project draws on seven datasets covering data center infrastructure, electricity generation/pricing, and thermoelectric water use across the US.

DatasetSourceGranularityCoverageAterio US Data CentersAterio (commercial)Facility-level~6,400 US data centers, point-in-time snapshot (Apr 2026)Avg. Retail Price of ElectricityEIA Electricity Data BrowserMonthly, by region2001–2026Net Generation, All FuelsEIA Electricity Data BrowserAnnual, by census division2001–2025Net Generation, Other RenewablesEIA Electricity Data BrowserAnnual, by census division2001–2025Electric Power Monthly, Table 3.7EIAState/sector, YoY2023–2024Generation Monthly (EIA-923)EIAMonthly, by state/producer/fuel2001–2026 (21 sheets)Thermoelectric Water Use EstimatesUSGSPlant-level, annual2008–2020

**Aterio US Data Centers Dashboard** Link: https://www.aterio.io/datasets/lst_us_data_centers

Aterio_US_Data_Centers_Dashboard.xlsx — Facility-level dataset of active, under-construction, and announced US data centers. Core table (US Data Centers Dataset, 42 cols) includes provider, state, facility/data-center square footage, total power (MW), and stage. Full field definitions are in the workbook's Data Dictionary sheet (lat/long, power capacity estimates, construction/activation dates, AI-facility flags, utility linkage). Remaining sheets are pre-built KPI/dashboard views, not raw data.

**Average Retail Price of Electricity** (https://www.eia.gov/electricity/data/browser/)

Average_retail_price_of_electricity.csv — Monthly retail electricity price (¢/kWh) by region and sector, Jan 2001–Jan 2026. Rows 1–4 are export metadata; header starts row 5 (description, units, source key, then one column per month).

**Net Generation for All Fuels** (Utility-Scale) Link: (https://www.eia.gov/electricity/data/eia923/)

Net_generation_for_all_fuels__utility-scale_.csv — Annual net generation (2001–2025) by US census division, all fuels combined. Same export format as above.

**Net Generation for Other Renewables** Link: https://www.eia.gov/electricity/data/eia923/ 

Net_generation_for_other_renewables.csv — Annual net generation (2001–2025) from non-hydro renewables (solar/wind/geothermal/biomass), by census division.

**EIA Electric Power Monthly** — Table 3.7 Link: https://www.eia.gov/electricity/monthly/ — go to "Table 3.7" under the monthly release; the full table index is here:

epa_03_07.xlsx — State/sector generation (thousand MWh), 2024 vs. 2023, with YoY % change. Merged multi-row header; data starts row 6. (Filename says "epa" but this is an EIA table — verify before formal citation.)

**Generation Monthly** (EIA-923) Link: https://www.eia.gov/electricity/data/eia923/ 

generation_monthly.xlsx — 21 sheets (one per year-range, 2001–2026), each with identical schema: YEAR, MONTH, STATE, TYPE OF PRODUCER, ENERGY SOURCE, GENERATION (Megawatthours). Most granular supply-side source in this project.

**Thermoelectric Water Use Estimates** Link: https://www.usgs.gov/data/thermoelectric-power-condenser-duty-estimates-month-and-cooling-type-use-calculate-water-use

published_annual_thermoelectric_water_use_estimates_2008-2020.csv — Plant-level modeled water withdrawal/consumption (wd_mgd, cu_mgd, with upper/lower bounds), 2008–2020, joined to plant name, county, state, water source, and cooling type.

## Data Preprocessing
Preprocessing

Six intermediate/output CSVs were built from the seven raw sources listed above.

1. Electricity prices → electricity_prices_states.csv


Dropped the 4-row metadata header from the raw EIA export.
Aggregated monthly prices to yearly to match the annual granularity of the other datasets.
Split the compound description field into state_or_region and sector (commercial / industrial).
Filtered down to state-level rows only (regional/national aggregates excluded).


2. Generation data → renewables_clean.csv, total_generation_clean.csv


Unpivoted (wide → long) the year columns (2001–2025) from Net_generation_for_other_renewables.csv and Net_generation_for_all_fuels__utility-scale_.csv into a single year column paired with renewable_generation / total_generation.
Retained units and source_key (EIA series ID) for traceability.
Split state_or_region and sector out of the description field, same as step 1.


3. Data centers → data_centers_facility_clean.csv


From the Aterio workbook, kept only three fields per facility: estimated power capacity, selected power capacity, and facility count (via state_or_region), alongside id, stage, state code, sector, and year.
Data reflects 2025 — the most current snapshot available at build time.


4. State-level rollup → data_center_energy_master_2025.csv


Grouped data_centers_facility_clean by state: summed estimated/selected power, counted facilities.
Joined to electricity_prices_states and the two generation files, all filtered to 2025, commercial sector.
Computed renewable_share = renewable_generation / total_generation.
Inner join drops any state missing a 2025 commercial-sector price/generation match.


5. Final master table → master_df_final.csv


Same as step 4, plus total_wd_mgd (thermoelectric water withdrawal), joined from published_annual_thermoelectric_water_use_estimates_2008-2020.csv.
Water data uses 2020 — the most recent year available in that source, vs. 2025 for the data-center fields.


⚠️ Known limitation: year mismatch

master_df_final.csv pairs 2025 data center capacity/counts with 2020 water withdrawal figures for the same state — a ~5-year gap. This was a deliberate call (2020 was the latest available thermoelectric data), not an error, but it means any correlation drawn between data-center power and water use should be read with that gap in mind — states with fast data-center growth since 2020 (e.g. Virginia, Texas, Arizona) may show a weaker or stronger relationship than the numbers alone suggest.

# Basic repository structure
``` bash 
├── data
│   ├── interim
│   ├── processed
│   ├── raw
├── notebooks
├── outputs
│   ├── figures
│   ├── tables
├── scripts
├── data_
├── LICENSE
├── README.md
```

# Results and evaluation
Given the limitations, no realistic conclusions can be drawn from this project, and is mainly an exploratory, skill-building exercise. The original idea for this project sought to answer these questions: How does sample data center power demand vary across states in relation to renewable electricity generation share, electricity prices, and thermo-electric water withdrawals? 
Finding: 
Subquestions: Which states have the highest sampled data center power capacity? Virginia, Arizona, and Georgia
Do states with larger sample data center power capacity tend to have lower or higher retail electricity prices? I observed a negative correlation between these two variables which is unexpected and may reflect the limitations. Typically data center power demand places more strain on the grid, draining supply and therefore increasing the price of retail electricity. 
Are states with larger electricity systems also the states with greater sample data center power capacity? Data Center power capacity and total generation do not have a strong correlation of .05. Electricty price and total power generation are the two variables with the strongest correlation in the analysis at .73. The two seem to be driven by a confounding variable, state size. 

[![Click to view Interactive Plot](Scatter_plot_1.png)](https://Ferry-17.github.io/Data-Center-Renewables-Exploration/outputs/figures/Scatter_plot_1.html)
[![Static Scatter plot 2](Data-Center-Renewables-Exploration/outputs/figures/Scatter_plot_2.png)
[![Click to view Interactive Plot](Scatter_plot_3.png)](https://Ferry-17.github.io/Data-Center-Renewables-Exploration/outputs/figures/Scatter_plot_3.html)
[![Click to view Interactive Plot](Scatter_plot_4.png)](https://Ferry-17.github.io/Data-Center-Renewables-Exploration/outputs/figures/Scatter_plot_4.html)

# Future work
Get access to more data and apply the same workflow. Get geospatial data relating to the variables and create interactive maps showing the relationships between them on the grid. 


# Limitations 
**Temporal misalignment (2025 vs. 2020).** Data center capacity and electricity/generation figures reflect 2025, while thermoelectric water withdrawal reflects 2020 — the most recent year available from USGS at time of analysis. This ~5-year gap means the water-use figures don't capture any data-center-driven demand growth since 2020. States with rapid data center buildout in that window (e.g. Virginia, Texas, Arizona) may show a weaker apparent relationship between power capacity and water use than actually exists today. Any correlation reported should be read as a **lower bound on the current relationship**, not a precise current-state measurement.

**Inner-join state coverage (30 of ~50 states).** Because the pipeline requires a complete match across data center, price, generation, *and* water data, only 30 states appear in the final table. Only 30 data center observations are shown, the free sample data set originally has 42, as I aggregated the corresponding data center variables in order to create a clean master table joined by state. States excluded are disproportionately those with no data center presence and/or missing generation data for 2025 — meaning the sample is not a random subset of US states, but skewed toward states already active in data center development. Findings should not be generalized to all 50 states without accounting for this selection.

**Correlation, not causation.** Any observed relationship between data center power capacity and water withdrawal reflects state-level co-location of power-intensive industries and thermoelectric generation — it does not establish that data centers *cause* water withdrawal, since thermoelectric plants serve many consumers beyond data centers.pwd
