# Data

| File | What it is | Created by |
|---|---|---|
| `esg_country_year_2000_2020.csv` | Modelling table: **4,011 rows** (191 countries × 2000–2020), one row per country-year, 70 indicators + region + income group + target | Notebook Section 2 |
| `esg_data_dictionary.csv` | Description, World Bank indicator code and unit of every column | Notebook Section 2 |
| `WB_ESG.csv` *(not in the repo)* | Raw long-format file, ~185 MB, 461,719 rows | Downloaded automatically from Data360 |

## Source and licence

- **Dataset:** World Bank Data360, *Environment, Social & Governance (ESG)* database, ID `WB_ESG`
- **Page:** https://data360.worldbank.org/en/dataset/WB_ESG
- **Direct download:** https://data360files.worldbank.org/data360-data/data/WB_ESG/WB_ESG.csv
- **Country classification (region, income group):** World Bank API, https://api.worldbank.org/v2/country
- **Licence:** Creative Commons Attribution 4.0 (CC BY 4.0). Redistribution with attribution is allowed, so the processed table is included here.
- **Date accessed:** October 2026

## How the processed table was built

1. Keep years 2000–2020.
2. Pivot from long format (country × indicator × year) to one row per country-year.
3. Remove ~45 regional/income aggregates ("World", "High income", …) using the World Bank country API (aggregates have region `NA`).
4. Add region and income group.
5. Keep rows with a CO₂-per-person value; rename indicator codes to short readable names.

To rebuild from scratch, delete `esg_country_year_2000_2020.csv` and run the notebook; Section 2 downloads the raw file and recreates it.
