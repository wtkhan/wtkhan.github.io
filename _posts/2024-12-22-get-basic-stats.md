---
layout: post
title: "Get basic statistics from a large spreadsheet in under five minutes"
summary: "Drunk on the power of analytical databases"
description: " "
---

I am always on the lookout for command-line tools that can get me from a raw data download to some basic numbers as quickly as possible. With [DuckDB](https://duckdb.org/docs/api/cli/overview.html) and its [aggregate functions](https://duckdb.org/docs/sql/functions/aggregates.html), I may have found my new default flow.

Below I query a salary [spreadsheet](https://www.dol.gov/agencies/eta/foreign-labor/performance) for summary statistics for data science positions filled in the past quarter. I trust this data from the federal government more than job postings with wide ranges and self-reported surveys on Levels.FYI and elsewhere. The main flaw with relying on this data is that they only report base pay, which can be the smaller part of a total compensation package for some companies and positions

Before DuckDB, I would have to use pandas's `read_excel()`, which takes a good bit of time parsing the text-heavy columns before loading the entire dataset into memory. DuckDB not only loads the data faster into memory, but I could have continued to query the spreadsheet itself without as much lag.
```sql
-- Data is 2024Q4 filings
.mode markdown
install spatial;
load spatial;
create or replace table df
    select *
    from st_read('Downloads/PERM_Disclosure_Data_FY2022_Q4.xlsx');
summarize(
    select
        coalesce(wage_offer_to, wage_offer_from) as base_pay
    from df
    where job_title ilike '%data scientist%'
);
```

In case you're interested, this produces the following output:
| column_name | column_type | min  |   max    | approx_unique |        avg         |        std        |        q25         |        q50         |        q75         | count | null_percentage |
|-------------|-------------|------|----------|---------------|--------------------|-------------------|--------------------|--------------------|--------------------|-------|-----------------|
| base_pay    | DOUBLE      | 73.0 | 650000.0 | 452           | 152044.49745940792 | 46640.04639860068 | 125499.03596938777 | 155487.80074193547 | 172934.37240649346 | 1047  | 0.0%            |

