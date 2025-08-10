---
layout: post
title: "Developing in SQL with dummy tables and recursive CTEs"
summary: "Some powerful tools for high-impact queries with confidence"
description: " "
---

While unit testing SQLAlchemy and compiled SQL logs are great "formal" testing tools, occasionally I need something more ad hoc to learn/debug specific transforms. Whether I am whipping up spaghetti SQL or reviewing somebody else's, it's helpful to create dummy tables to step through parts of logic. This post demonstrates how I mocked up data to learn to use (and appreciate) recursive CTEs.

## Scenario: Sessionizing API requests

The following is a basic early-stage product analytics need. Imagine you have a dump of API requests and have to sessionize them by account/user. This is to enable stakeholders to query the requests data to monitor usage metrics.

### Logic

Queries below are in DuckDB, my favorite in-memory analytical database flavor. As with any nonstandard SQL work, actual implementation will vary by vendor.

In a few places, I used ChatGPT (specifically the free 4o mini model) to generate formatted SQL and then manually reviewed them. ChatGPT doesn't have a great command of DuckDB syntax yet, so I ran each chunk and made necessary corrections and fed them back into the exchange.

<!-- TODO Pick back up on creating more 'deterministic' test cases where we can specify a handful of requests at certain times -->
```sql
-- create the table
create or replace table api_requests as
select
    -- random user ID between 1 and 10
    ((random() * 9) + 1)::int as user_id,
    -- random endpoint ID between 1 and 100
    concat('api.google.com/v1/', ((random()*4) + 1)::int::varchar) as endpoint,
    -- generate random timestamps over a period of time (e.g., a 1-day period)
    '2024-12-01 00:00:00'::timestamp + interval (floor(random() * 720)) minute
        -- generate random minutes from 0 to 720 (1 day = 1440 minutes)
        as request_timestamp
from generate_series(1, 100)
order by user_id, request_timestamp;
```

Let's assume a user session is defined as API requests made within 20 minutes of each other. If we wanted the start and end of a session each day, then for each user, the strategy is to:

* Flag the first timestamps that mark the start of a new session
* Count the number of new session flags
* For each new session, get the earliest and latest timestamps

```sql
with session_flags as (
    select
        *,
        coalesce(cast(datediff('minute', lag(request_timestamp) over(partition by user_id order by request_timestamp rows between unbounded preceding and current row), request_timestamp) > 20 as int), 1) as new_session
    from api_requests
)
, session_nums as (
select
    user_id,
    request_timestamp,
    sum(new_session) over(partition by user_id order by request_timestamp rows between unbounded preceding and current row) as session_num
from session_flags
)
select
    user_id,
    session_num,
    min(request_timestamp) over(partition by user_id, session_num) as started_at,
    max(request_timestamp) over(partition by user_id, session_num) as ended_at
from session_nums
order by user_id, request_timestamp;
```

Now suppose that in addition to counts of sessions, we also want to know how each user moved between endpoints and learn about any common flow patterns. The challenge is that users have varying sets of endpoints. If we tried to self-join the API requests table, then we'd have to manually join each request. Happily, recursive CTEs can help:

```sql
with recursive rsessions as (
    with session_flags as (
    select
        *,
        coalesce(cast(datediff('minute', lag(request_timestamp) over(partition by user_id order by request_timestamp rows between unbounded preceding and current row), request_timestamp) > 20 as int), 1) as new_session
    from api_requests
    )
    , session_nums as (
        select
            *,
            sum(new_session) over(partition by user_id order by request_timestamp rows between unbounded preceding and current row) as session_num
        from session_flags
    )
    , start_and_end_times as (
        select
            user_id,
            request_timestamp,
            endpoint,
            min(request_timestamp) over(partition by user_id, session_num) as started_at,
            max(request_timestamp) over(partition by user_id, session_num) as ended_at
        from session_nums
        order by user_id, request_timestamp
    )
    -- base case
    select
        user_id,
        request_timestamp,
        started_at,
        ended_at,
        [endpoint] as flow,
        1 as session_num
    from start_and_end_times
    where session_num = 1
    union all
    select
        rs.user_id,
        rs.request_timestamp,
        rs.started_at,
        rs.ended_at,
        array_append(rs.flow, endpoint),
        s.session_num,
    from rsessions rs
    join start_and_end_times s
        on s.user_id = rs.user_id
        and rs.session_num + 1 = s.session_num
    where len(rs.flow) < (select max(session_num) from session_nums)
)
select *
from rsessions
limit 10;
```

## Comparing runtimes

---
Given the `api_requests` table and the data, we could create a problem scenario where recursive Common Table Expressions (CTEs) are helpful for identifying and grouping API request sessions.

### Problem Scenario:
We want to identify sessions for each account. A "session" is defined as a group of requests made by the same account where the requests are within 30 minutes of each other. If there's a gap of more than 30 minutes between two consecutive requests, a new session starts. The challenge is to identify these session groups.

#### Columns Required for the Problem:
1. **user_id**: The account making the request.
2. **request_timestamp**: The timestamp of the API request.
3. **session_id**: A unique identifier for each session.
4. **session_start_time**: The start time of the session.
5. **session_end_time**: The end time of the session.

---

### Plan:
1. **Sort the Requests**: First, we'll need to order the requests by account and timestamp so that we can track whether there is a gap between consecutive requests.
2. **Identify Session Gaps**: For each request, we'll check if the gap from the previous request for the same account is greater than 30 minutes. If it is, it indicates a new session.
3. **Group Requests into Sessions**: Use a recursive CTE to assign requests to sessions, keeping track of the `session_id` for each account.
4. **Aggregate Session Information**: Finally, we can retrieve the session start and end times, along with the session ID for each account.

---

### Step-by-Step SQL Implementation:

1. **Create a session identifier** for each request. This is done by comparing each request's timestamp with the previous request's timestamp. If the gap is greater than 30 minutes, it's the start of a new session.

2. **Recursive CTE to Track Sessions**: We’ll use the recursive CTE to propagate the session identifier across the dataset.

---

### SQL Query:

```sql
WITH RECURSIVE sessions AS (
    -- Base case: The first request for each account starts a session
    SELECT
        user_id,
        request_timestamp,
        request_timestamp AS session_start_time,
        request_timestamp AS session_end_time,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY request_timestamp) AS row_num,
        1 AS session_id -- Initial session for each account
    FROM api_requests
    WHERE request_timestamp = (SELECT MIN(request_timestamp) FROM api_requests)

    UNION ALL

    -- Recursive case: Check for gaps and continue the session if the gap is within 30 minutes
    SELECT
        ar.user_id,
        ar.request_timestamp,
        -- If the gap is greater than 30 minutes, start a new session
        CASE
            WHEN ar.request_timestamp > (s.session_end_time + INTERVAL '30 minutes')
            THEN ar.request_timestamp
            ELSE s.session_start_time
        END AS session_start_time,
        -- Always update the session end time
        ar.request_timestamp AS session_end_time,
        ROW_NUMBER() OVER (PARTITION BY ar.user_id ORDER BY ar.request_timestamp) AS row_num,
        CASE
            WHEN ar.request_timestamp > (s.session_end_time + INTERVAL '30 minutes')
            THEN s.session_id + 1
            ELSE s.session_id
        END AS session_id
    FROM
        api_requests ar
    JOIN
        sessions s
    ON
        ar.user_id = s.user_id AND ar.request_timestamp > s.request_timestamp
)
select * from sessions order by user_id, request_timestamp;
-- Select the final result
SELECT
    user_id,
    session_id,
    MIN(session_start_time) AS session_start_time,
    MAX(session_end_time) AS session_end_time
FROM sessions
GROUP BY user_id, session_id
ORDER BY user_id, session_start_time;
```

### Explanation:

1. **Base Case**: The first request for each account starts a new session. This is the initial row in the recursive query.
2. **Recursive Case**: The recursive part checks for each subsequent request. If the gap between the current request and the previous one is greater than 30 minutes, it assigns a new session ID. Otherwise, it continues the current session.
3. **Final Output**: The final query aggregates the session data, returning the session ID, start time, and end time for each session.

### Expected Output:

Based on the data in the `api_requests` table, the query would return something like:

| user_id | session_id | session_start_time     | session_end_time       |
|------------|------------|------------------------|------------------------|
| 1          | 1          | 2024-11-01 08:00:00    | 2024-11-01 09:00:00    |
| 1          | 2          | 2024-11-01 09:20:00    | 2024-11-01 10:00:00    |
| 2          | 1          | 2024-11-01 08:00:00    | 2024-11-01 09:30:00    |
| 2          | 2          | 2024-11-01 09:45:00    | 2024-11-01 10:20:00    |

### Use Case:
This query is helpful for sessionizing API requests by account, especially in cases where you need to analyze user behavior over time, track sessions, or calculate metrics like session duration, active time, and engagement patterns. Recursive CTEs make it easy to dynamically group requests that belong to the same session based on the 30-minute gap rule.

```sql
-- Create the table
CREATE OR REPLACE TABLE api_requests (
    user_id INT,
    request_timestamp TIMESTAMP
);

-- Insert 100 rows of mock data
INSERT INTO api_requests (user_id, request_timestamp)
SELECT
    -- Random user_id between 1 and 10
    FLOOR(RAND() * 10) + 1 AS user_id,

    -- Generate random timestamps over a period of time (e.g., a 1-day period)
    DATEADD(MINUTE,
        FLOOR(RAND() * 1440),  -- Generate random minutes from 0 to 1439 (1 day = 1440 minutes)
        '2024-11-01 00:00:00'::TIMESTAMP
    ) AS request_timestamp
FROM TABLE(GENERATOR(ROWCOUNT => 100));
```
