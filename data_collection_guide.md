# Data Collection Guide: State-wise Budget Allocation vs Outcomes (India, 2018-2023)

Beginner-friendly. Use this to collect the REAL data. (The CSV I generated is synthetic and only for practice.)

> Official websites rename tables and revise numbers. If a table name below is not found, search the website for the **indicator name** (e.g. "IMR state-wise SRS"). Always note the **publication year and table name** in your report.

---

## 1. Indicators to collect

### Inputs (spending, from state budgets)
| Indicator | Definition | Unit |
|---|---|---|
| health_spend | State government expenditure on "Health and Family Welfare" | Rs crore |
| edu_spend | State government expenditure on "Education, Sports, Art and Culture" | Rs crore |
| infra_spend | State expenditure on roads and bridges, irrigation, energy, urban/rural development (decide and **write down** which heads you include) | Rs crore |
| population | Mid-year population estimate (used to make per-capita values) | lakh |

Per-capita spending is **calculated by you** (see code below), not downloaded.

### Outcomes
| Indicator | Definition | Unit |
|---|---|---|
| imr | Infant deaths (under 1 year) per 1,000 live births | per 1,000 |
| life_expectancy | Expected years of life at birth | years |
| literacy | Percent of people aged 7+ (or 15+) who can read and write | % |
| enrolment | School enrolment: Gross Enrolment Ratio (choose one level, e.g. secondary, and keep it fixed) | % |
| dropout | School dropout rate (same level as enrolment) | % |
| electricity_access | Households with electricity | % |
| gsdp_pc | GSDP (or NSDP) per person, current prices | Rs |

---

## 2. Where to find each indicator

| Indicator | Best official source | Notes |
|---|---|---|
| health_spend, edu_spend, infra_spend | **RBI: "State Finances: A Study of Budgets"** and **RBI Handbook of Statistics on Indian States** (expenditure by function/sector) | Annual, state-wise. Most reliable for spending. State budget documents and CAG state finance reports can fill gaps. |
| population | **Census 2011 + official projections** (National Commission on Population / MoHFW projections). RBI Handbook also lists population. | Census is every 10 years, so 2018-2023 values are **projections**. |
| imr | **SRS Bulletins / Statistical Reports (Registrar General of India)**, also **NITI Aayog SDG India Index** and **NFHS-5** | SRS is annual and state-wise (best for a time series). NFHS-5 (2019-21) gives one period value only. |
| life_expectancy | **SRS abridged life tables** (3-year rolling averages) / population projection reports | Rolling averages, so treat as approximate yearly values. |
| literacy | **Census 2011**, **NSS 75th round (2017-18)**, **PLFS** annual reports; NFHS-5 for adult literacy by sex | Not available every year from one source. Choose one source and **say so**; interpolate between survey years and explain it. |
| enrolment, dropout | **UDISE+** (Unified District Information System for Education Plus), Ministry of Education | Annual, state-wise GER/NER and dropout rates by level. |
| electricity_access | **NFHS-5** (households with electricity), **NITI Aayog SDG India Index** (SDG 7), Saubhagya portal | NFHS is periodic. SDG Index gives some intermediate years. |
| gsdp_pc | **RBI Handbook of Statistics on Indian States** (GSDP/NSDP and per-capita), **MoSPI** state domestic product | Choose current prices (not constant) and **say so**. |
| Anything else / backup | **data.gov.in** (Open Government Data platform) | Search by indicator and state. Good for CSV downloads. |

**Plan for gaps:** survey sources (NFHS, Census) do not cover every year. Options: use a single consistent source; interpolate between survey years; or use only the years that exist. Explain your choice in "Limitations".

---

## 3. CSV template

Save as `state_data_raw.csv`. One row per state per year (28 states x 6 years = 168 rows).

```csv
state,year,population,health_spend,edu_spend,infra_spend,imr,life_expectancy,literacy,enrolment,dropout,electricity_access,gsdp_pc
Kerala,2018,350.0,9100.0,21000.0,16500.0,10.0,75.0,94.0,95.0,3.0,100.0,235000
Bihar,2018,1180.0,8500.0,39000.0,34000.0,38.0,68.5,62.0,66.0,25.0,97.0,45000
```

*(The two rows above are format examples with made-up numbers. Replace with real values.)*

### Data dictionary

| Column | Type | Unit | Description | Source | Allowed range (for checking) |
|---|---|---|---|---|---|
| state | text | none | State name, standardised (e.g. "Odisha", not "Orissa") | all | 28 unique names |
| year | integer | calendar/financial year | Use one rule: financial year start (2018 = 2018-19) | all | 2018-2023 |
| population | number | lakh | Mid-year population estimate | Census projections | > 0 |
| health_spend | number | Rs crore | State expenditure on health | RBI State Finances | > 0 |
| edu_spend | number | Rs crore | State expenditure on education | RBI State Finances | > 0 |
| infra_spend | number | Rs crore | State expenditure on infrastructure heads you chose | RBI State Finances | > 0 |
| imr | number | per 1,000 live births | Infant mortality rate | SRS / NFHS-5 | 0-100 |
| life_expectancy | number | years | Life expectancy at birth | SRS life tables | 55-85 |
| literacy | number | % | Literacy rate | Census / NSS / PLFS | 0-100 |
| enrolment | number | % | Gross enrolment ratio (fixed level) | UDISE+ | 0-130 (GER can exceed 100) |
| dropout | number | % | Dropout rate (same level as enrolment) | UDISE+ | 0-100 |
| electricity_access | number | % | Households with electricity | NFHS-5 / SDG Index | 0-100 |
| gsdp_pc | number | Rs per person | GSDP per capita, current prices | RBI Handbook / MoSPI | > 0 |

Derived columns (calculated in code): `health_pc`, `edu_pc`, `infra_pc` = spending per person in Rs.

---

## 4. Python (pandas) code: merge, standardise names, handle missing values, per-capita

Every step is explained in comments. Put each source in its own CSV first (one file per source), each with the columns `state, year` plus its own indicators.

```python
import pandas as pd
import numpy as np

# ---------------------------------------------------------------
# STEP 1: Read each source file into its own table (DataFrame)
# ---------------------------------------------------------------
budget   = pd.read_csv("rbi_budget.csv")        # state, year, health_spend, edu_spend, infra_spend, gsdp_pc
pop      = pd.read_csv("population.csv")        # state, year, population
health   = pd.read_csv("srs_health.csv")        # state, year, imr, life_expectancy
school   = pd.read_csv("udise.csv")             # state, year, enrolment, dropout
literacy = pd.read_csv("literacy.csv")          # state, year, literacy
power    = pd.read_csv("electricity.csv")       # state, year, electricity_access

# ---------------------------------------------------------------
# STEP 2: Standardise state names BEFORE merging.
# Different sources spell states differently ("Orissa" vs "Odisha").
# If names do not match exactly, the merge silently drops rows!
# ---------------------------------------------------------------
name_fix = {
    "Orissa": "Odisha",
    "Uttaranchal": "Uttarakhand",
    "Jammu & Kashmir": "Jammu And Kashmir",
    "W. Bengal": "West Bengal",
    "A.P.": "Andhra Pradesh",
}

def clean_names(table):
    table["state"] = (table["state"]
                      .str.strip()          # remove extra spaces
                      .str.title()          # Same capital letters everywhere
                      .str.replace("&", "And", regex=False)
                      .replace(name_fix))   # fix known spelling variants
    return table

tables = [clean_names(t) for t in [budget, pop, health, school, literacy, power]]

# ---------------------------------------------------------------
# STEP 3: Merge all tables on state + year.
# how="left" keeps every row of the first table (budget) and
# adds columns from the others. Missing matches become NaN.
# ---------------------------------------------------------------
df = tables[0]
for t in tables[1:]:
    df = df.merge(t, on=["state", "year"], how="left")

print("Shape after merge:", df.shape)   # expect (168, 13)

# Check: which states did not match? (should be none missing)
print(df.groupby("state").size().describe())

# ---------------------------------------------------------------
# STEP 4: Look at missing values
# ---------------------------------------------------------------
print(df.isna().sum())                  # count of missing values per column

# ---------------------------------------------------------------
# STEP 5: Handle missing values.
# For data over time, fill gaps by interpolation WITHIN each state
# (a state's values change gradually). Sort first so years are in order.
# Do NOT average across states: Kerala and Bihar are very different.
# ---------------------------------------------------------------
df = df.sort_values(["state", "year"]).reset_index(drop=True)
num_cols = [c for c in df.columns if c not in ("state", "year")]

for c in num_cols:
    df[c] = df.groupby("state")[c].transform(lambda s: s.interpolate(limit_direction="both"))

# If a whole column is missing for a state, interpolation cannot help.
# Then report it honestly rather than inventing numbers.
print("Still missing:", df.isna().sum().sum())

# ---------------------------------------------------------------
# STEP 6: Check for impossible values using the allowed ranges
# ---------------------------------------------------------------
ranges = {"imr": (0, 100), "literacy": (0, 100), "dropout": (0, 100),
          "electricity_access": (0, 100), "life_expectancy": (55, 85)}
for col, (lo, hi) in ranges.items():
    bad = df[(df[col] < lo) | (df[col] > hi)]
    if len(bad):
        print(f"Check {col}: {len(bad)} rows out of range\n", bad[["state", "year", col]])

# ---------------------------------------------------------------
# STEP 7: Per-capita spending (Rs per person)
# spend is in Rs crore (1 crore = 10,000,000)
# population is in lakh (1 lakh = 100,000)
# so Rs per person = (spend * 10^7) / (population * 10^5) = spend / population * 100
# ---------------------------------------------------------------
for sector in ["health", "edu", "infra"]:
    df[f"{sector}_pc"] = df[f"{sector}_spend"] / df["population"] * 100

# ---------------------------------------------------------------
# STEP 8: Save the cleaned file
# ---------------------------------------------------------------
df.to_csv("state_data_clean.csv", index=False)
df.head()
```

### Common beginner mistakes
1. Merging before fixing state names, which silently loses rows.
2. Mixing units (lakh vs crore, or Rs vs Rs thousand). Write units in the data dictionary.
3. Mixing financial years and calendar years.
4. Filling missing values with the overall average instead of interpolating within each state.
5. Comparing total spending instead of per-capita spending.
6. Mixing different education levels (primary vs secondary) across years.
