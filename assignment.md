# Assignment

## Brief

Write the SQL statements for the following questions.

## Instructions

Paste the answer as SQL in the answer code section below each question.

### Question 1

Let's revisit our `Austin Bikeshare Dataset` dbt project. Modify the `dim_station.sql` model to include the following columns:

- `total_duration` (sum of `duration` for each station in seconds)
- `total_starts` (count of `start_station_name` for each station)
- `total_ends` (count of `end_station_name` for each station)

Then, rebuild the models with the following command to see if the changes are correct:

```bash
dbt run
```

Answer:

Paste the `dim_station.sql` model here:

```sql
{{ config(materialized='table') }}

with rides as (

    -- Austin Bikeshare: duration stored in minutes → convert to seconds
    select
        start_station_name,
        end_station_name,
        duration_minutes * 60 as duration_seconds
    from {{ ref('fact_trips') }}

),

starts as (

    select
        start_station_name as station_name,
        sum(duration_seconds) as total_duration_starts,
        count(*) as total_starts
    from rides
    where start_station_name is not null
    group by 1

),

ends as (

    select
        end_station_name as station_name,
        sum(duration_seconds) as total_duration_ends,
        count(*) as total_ends
    from rides
    where end_station_name is not null
    group by 1

),

stations as (

    select station_name from starts
    union distinct
    select station_name from ends

)

select
    s.station_name,

    coalesce(st.total_duration_starts, 0) + coalesce(en.total_duration_ends, 0) as total_duration,
    coalesce(st.total_starts, 0) as total_starts,
    coalesce(en.total_ends, 0) as total_ends

from stations s
left join starts st on s.station_name = st.station_name
left join ends   en on s.station_name = en.station_name
order by s.station_name;


```

## Submission

This assignment was completed with assistance from ChatGPT (OpenAI).
ChatGPT was used to:

Clarify dbt and SQL concepts

Generate example SQL structures

Provide debugging support for BigQuery and dbt

Explain and refine dimensional model logic

Support code correctness and best practices

All final implementation decisions and code validation were performed by me.
