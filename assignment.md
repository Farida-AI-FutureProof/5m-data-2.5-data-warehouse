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

    -- Austin Bikeshare stores duration in minutes; convert to seconds
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

    coalesce(st.total_duration_starts, 0)
      + coalesce(en.total_duration_ends, 0) as total_duration,

    coalesce(st.total_starts, 0) as total_starts,
    coalesce(en.total_ends, 0) as total_ends

from stations s
left join starts st on s.station_name = st.station_name
left join ends  en on s.station_name = en.station_name
order by s.station_name;



```

## Submission

Validation — Tested in Google Colab

To verify correctness, the SQL logic was replicated and tested in Google Colab using a bikeshare trips dataset containing:

start_station_name

end_station_name

duration_minutes

The following was validated:

✔ Duration converted to seconds
✔ Start-based and end-based aggregations match expected logic
✔ Union of stations produces a complete station dimension
✔ Final output includes station_name, total_duration, total_starts, total_ends
✔ Results correspond to realistic ride activity patterns

Code used for testing (summarized):
df['duration_seconds'] = df['duration_minutes'] * 60

starts = df.groupby('start_station_name').agg(
    total_duration_starts=('duration_seconds','sum'),
    total_starts=('start_station_name','count')
)

ends = df.groupby('end_station_name').agg(
    total_duration_ends=('duration_seconds','sum'),
    total_ends=('end_station_name','count')
)

dim_station = (
    stations
    .merge(starts, how='left', on='station_name')
    .merge(ends, how='left', on='station_name')
).fillna(0)

dim_station['total_duration'] = (
    dim_station['total_duration_starts']
    + dim_station['total_duration_ends']
)

The resulting table matched the output expected from the dbt model, confirming correctness.

Submission Notes

This assignment was completed with assistance from ChatGPT (OpenAI).
ChatGPT was used to:

Clarify dbt dimensional modelling concepts

Validate SQL logic

Assist with Colab-based data testing

Support debugging and verifying metric calculations

All final decisions, SQL writing, testing, and validation were performed independently.
