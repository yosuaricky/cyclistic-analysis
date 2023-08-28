# Data analysis on 'Cyclistic', a Chicago based bike-sharing company

## Introduction

This project is my submission for Study Case 1 on **[Google Data Analytics Professional Certificate](https://www.coursera.org/professional-certificates/google-data-analytics) - Capstone Project**.

Cyclistic is a company providing bike-share program that features more than 5,800 bicycles and 600 docking stations. Cyclistic sets itself apart by also offering reclining bikes, hand tricycles, and cargo bikes, making bike-share more inclusive to people with disabilities and riders who can’t use a standard two-wheeled bike. The majority of riders opt for traditional bikes; about 8% of riders use the assistive options. Cyclistic users are more likely to ride for leisure, but about 30% use them to commute to work each day.

## Scenario

You are a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives must approve your recommendations, so they must be backed up with compelling data insights and professional data visualizations.

## Data Analysis Process

In order to answer the key business questions, I will follow the steps of the data analysis process: _ask, prepare, process, analyze, share, and act_.

<details><summary>Phase 1 - Ask</summary>

### Phase 1 - Asking the right question

Based on above scenario, I started this project by asking the stakeholder about the project goals. This phase will give me clear understanding about the business tasks, and stay focus on the project goals. This phase will also help me to decide: what data should I get, and how to perform analysis on this kind of data.

The stakeholder has set a clear goal: **Converting casual riders into annual members**. In order to do that, first I need to find out how do annual members and casual riders use Cyclistic bikes differently?

</details>

<details><summary>Phase 2 - Prepare</summary>

### Phase 2 - Preparing the data

For the purposes of this case study, I will use [Divvy’s historical trip data](https://divvy-tripdata.s3.amazonaws.com/index.html), and download the last 12 months of trip data to analyze and identify trends. The data has been made available by Motivate International Inc. under this [license](https://ride.divvybikes.com/data-license-agreement). Because Cyclistic is a fictional company, I will use this public data to explore how different customer types are using their bikes. However, data-privacy issues prohibit me from using rider's personally identifiable information, so I won’t be able to connect pass purchases to credit card numbers to determine if casual riders live in the company service area or if they have purchased multiple single passes.

I downloaded all of the data and kept the original version on my google drive folder in order to use it for future reference.

<details>

<summary>Divvy's 2022 trip data</summary>
  
```
202201-divvy-tripdata.csv
202202-divvy-tripdata.csv
202203-divvy-tripdata.csv
202204-divvy-tripdata.csv
202205-divvy-tripdata.csv
202206-divvy-tripdata.csv
202207-divvy-tripdata.csv
202208-divvy-tripdata.csv
202209-divvy-tripdata.csv
202210-divvy-tripdata.csv
202211-divvy-tripdata.csv
202212-divvy-tripdata.csv
```

</details>

All of the data are on csv format and contains every record of user's trip data in 2022. I uploaded and imported all of the csv to BigQuery, and because each data contains equal column name, I combined them into one-big-table named `bike_trip_2022`.

<details>

<summary>Combine dataset</summary>

```sql
SELECT * FROM `utopian-saga-394613.cyclistic_data.m01_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m02_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m03_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m04_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m05_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m06_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m07_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m08_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m09_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m10_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m11_2022`
UNION ALL
SELECT * FROM `utopian-saga-394613.cyclistic_data.m12_2022`
```

</details>

Table schema in `bike_trip_2022`:

| Field name         | Type      |
| ------------------ | --------- |
| ride_id            | STRING    |
| rideable_type      | STRING    |
| started_at         | TIMESTAMP |
| ended_at           | TIMESTAMP |
| start_station_name | STRING    |
| start_station_id   | STRING    |
| end_station_name   | STRING    |
| end_station_id     | STRING    |
| start_lat          | FLOAT     |
| start_lng          | FLOAT     |
| end_lat            | FLOAT     |
| end_lng            | FLOAT     |
| member_casual      | STRING    |

Identify total records in `bike_trip_2022` for data cleaning:

```sql
SELECT
  COUNT(*) AS total_records
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022_v1`
```

| total_records |
| ------------- |
| 5667717       |

Checking for duplicates:

```sql
SELECT
  COUNT(DISTINCT ride_id) AS unique_records
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022_v1`
```

| unique_records |
| -------------- |
| 5667717        |

The total of unique records is equal to total records, so I can confirm there is no duplicate in dataset. However, after further inspection, I found problems in the data:

- `member_casual` is ambiguous, there must be a better name for it
- There are NULL values recorded
- Timestamp in `ended_at` are recorded earlier than `started_at`
- Lots of trips duration is occured under 10 seconds

</details>

<details><summary>Phase 3 - Process</summary>

### Phase 3 - Processing the data

To make it easier on analyzing the data, I made a couple of changes:

- Create a column called `ride_length` to calculate the length of each ride by subtracting the column `started_at` from the column `ended_at`
- Create query using `CASE` to identify the day of the week on each trip, by extracting the date part from column `started_at` and return the results on HH:MM:SS format
- Change column `member_casual` to `user_type` because it's more self explanatory

```sql
SELECT
  ride_id,
  rideable_type,
  started_at,
  ended_at,
  datetime_diff(ended_at, started_at, MINUTE) AS ride_length,
  (
  SELECT
    CASE
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 1 THEN 'Sunday'
      WHEN EXTRACT(DAYOFWEEK
    FROM
      started_at) = 2 THEN 'Monday'
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 3 THEN 'Tuesday'
      WHEN EXTRACT(DAYOFWEEK
    FROM
      started_at) = 4 THEN 'Wednesday'
      WHEN EXTRACT(DAYOFWEEK FROM started_at) = 5 THEN 'Thursday'
      WHEN EXTRACT(DAYOFWEEK
    FROM
      started_at) = 6 THEN 'Friday'
    ELSE
    'Saturday'
  END
    ) AS day_of_week,
  start_station_name,
  start_station_id,
  end_station_name,
  end_station_id,
  start_lat,
  start_lng,
  end_lat,
  end_lng,
  member_casual as user_type
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022`;
```

From the query above, I created new table named `bike_trip_2022_v1` for the convenience in data cleaning process.

Table schema in `bike_trip_2022_v1`:

| Field name         | Type      |
| ------------------ | --------- |
| ride_id            | STRING    |
| rideable_type      | STRING    |
| started_at         | TIMESTAMP |
| ended_at           | TIMESTAMP |
| ride_length        | INTEGER   |
| day_of_week        | STRING    |
| start_station_name | STRING    |
| start_station_id   | STRING    |
| end_station_name   | STRING    |
| end_station_id     | STRING    |
| start_lat          | FLOAT     |
| start_lng          | FLOAT     |
| end_lat            | FLOAT     |
| end_lng            | FLOAT     |
| member_casual      | STRING    |

Based on problems I found earlier in dataset, I do the following:

- Removed all rows with NULL values, because it could impact result of the analysis
- Removed all rows with faulty recorded timestamp
- Removed all rows with trip durations under 1 minute

```sql
SELECT
  *
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022_v1`
WHERE
  start_station_name IS NOT NULL
  AND end_station_name IS NOT NULL
  AND start_station_id IS NOT NULL
  AND end_station_id IS NOT NULL
  AND start_lat IS NOT NULL
  AND start_lng IS NOT NULL
  AND end_lat IS NOT NULL
  AND end_lng IS NOT NULL
  AND ended_at > started_at
  AND ride_length > 0
```

I saved the result into new table named `bike_trip_2022_v2`, and check the total records to identify whether the data is sufficient enough for analysis or not, compared to total records in the dirty dataset.

```sql
SELECT
  COUNT(*)
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022_v2`
```

| v2_records |
| ---------- |
| 4292709    |

24% records deleted from the dirty dataset, and remaining 76% of data is sufficient for the next phase.

</details>

<details><summary>Phase 4 - Analyze</summary>

### Phase 4 - Analyzing the data

First, in order to gathered summary from the data, I created sql queries to:

- Calculate the maximum duration of `ride_length`
- Calculate the average duration of `ride_length`
- Calculate the minimum duration of `ride_length`

```sql
SELECT
  -- calculate maximum trip duration
  MAX(ride_length) AS longest_trip,
  -- calculate mean of trip duration, and rounded the result
  ROUND(AVG(ride_length), 2) AS average_trip,
  -- calculate minimum trip duration
  MIN(ride_length) AS shortest_trip
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022_v2`
```

The result of summary:
| longest_trip | average_trip | shortest_trip |
| ------- | ------- | ------- |
| 34354 | 16.9 | 1 |

And then I start analyzing the data to find out:

- What is the percentage of member and casual user from total user:

```sql
SELECT
  user_type,
  -- find the percentage of member and casual user from total user and round the result
  ROUND(COUNT(*) / (SELECT COUNT(*) FROM `utopian-saga-394613.cyclistic_data.bike_trip_2022_v2`) * 100, 1) AS user_percentage
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022_v2`
GROUP BY
  user_type
```

result:
| user_type | user_percentage |
| ------- | ------- |
| member | 59.7 |
| casual | 40.3 |

- What is the total between each type of bike from the `rideable_type`:

```sql
SELECT
  rideable_type,
  COUNT(*) AS total
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022_v2`
GROUP BY
  rideable_type
ORDER BY
  total desc
```

result:
| rideable_type | total |
| ------- | ------- |
| classic_bike | 2558903 |
| electric_bike | 1560462 |
| docker_bike | 173344 |

- Find the total between member and casual user on each month:

```sql
SELECT
  -- extract month from the date of trip
  EXTRACT(month
  FROM
    started_at) AS month,
  -- return 1 if user_type is 'member', then apply sum function for all returned value to find out the total of member on each month
  SUM(CASE
      WHEN user_type = 'member' THEN 1
    ELSE
    0
  END
    ) AS member,
  -- return 1 if user_type is 'casual', then apply sum function for all returned value to find out the total of casual-user on each month
  SUM(CASE
      WHEN user_type = 'casual' THEN 1
    ELSE
    0
  END
    ) AS casual
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022_v2`
GROUP BY
  month
ORDER BY
  month
```

result:
| month | member | casual |
| ------- | ------- | ------- |
| 1 | 66575 | 12481 |
| 2 | 72683 | 14973 |
| 3 | 146497 | 66409 |
| 4 | 177723 | 90816 |
| 5 | 277162 | 216938 |
| 6 | 322256 | 287553 |
| 7 | 324307 | 306612 |
| 8 | 328577 | 265748 |
| 9 | 307844 | 217485 |
| 10 | 257460 | 148865 |
| 11 | 178766 | 72366 |
| 12 | 101634 | 30979 |

- Find the total between member and casual user on each day_of_week:

```sql
SELECT
  day_of_week,
  -- return 1 if user_type is 'member', then apply sum function for all returned value to find out the total of member on each day_of_week
  SUM(CASE
      WHEN user_type = 'member' THEN 1
    ELSE
    0
  END
    ) AS member,
  -- return 1 if user_type is 'casual', then apply sum function for all returned value to find out the total of casual-user on each day_of_week
  SUM(CASE
      WHEN user_type = 'casual' THEN 1
    ELSE
    0
  END
    ) AS casual
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022_v2`
GROUP BY
  day_of_week
```

result:
| day_of_week | member | casual |
| ------- | ------- | ------- |
| Thursday | 408198 | 226566 |
| Tuesday | 403846 | 193413 |
| Wednesday | 405146 | 200515 |
| Monday | 368255 | 207540 |
| Saturday | 331315 | 361609 |
| Friday | 353087 | 244983 |
| Sunday | 291637 | 296599 |

</details>

<details><summary>Phase 5 - Share</summary>

### Phase 5 - Sharing insights

After creating summary and all metrics to answer the key business questions, I decided to create an interactive dashboard using Google Looker Studio.

![cyclistic dashboard](/project-image/cyclistic_dashboard.png)

This tool allow me to publicly share my insight, the dashboard will be able to filter data based on specific criteria such as:

- Filter data by range of date (01 January 2022 - 31 December 2022)
- Filter data based on type of user (member, casual)
- Filter data based on type of bike (classic, bike, electric bike, docked bike)

Feel free to interact with the dashboard by accessing this link:

**[Cyclistic Dashboard](https://lookerstudio.google.com/reporting/cf9914e4-e461-4ded-bdb3-d90bd6dcfc93)**

<sub>_because the large amount of data being generated, you may encounter slight delay when interacting with the dashboard_</sub>

</details>

<details><summary>Phase 6 - Act</summary>

### Phase 6 - Take action

After created the dashboard and interacted with it, I found couples of important insight that can drive bussiness marketing decisions:

#### Key findings:
- Because of summer season on United States, the peak of total user is in June - August. Total user will start decline gradually on winter season in September through January.
- Annual Cyclistic member use bike mostly during weekdays for daily activities such as going to work, shopping, going to school, etc.
- Casual user use bike mostly on weekend for their leisures time and physical exercises during holiday.
- The most favourite type of ride is classic bike, because classic bike offer more physical exercise than electric bike and user tend to appreciate more healthy lifestyle. 

What kind of action that Cyclistic can take to convert more casual user into annual member?

#### Recommendations:
- Marketing campaign should be targeted towards summer when users more likely to utilize Cyclistic service.
- The marketing team should promoting Cyclistic on Friday through Sunday, because casual user seen more on weekend during their leisures time.
- In order to gain new potential user, the marketing team can also consider to create advertising on every store that sell healthy product or facility like pharmacy, gym, and hospital.

</details>
