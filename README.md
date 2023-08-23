# My data analysis on 'Cyclistic', a fictional bike-sharing company.

## Introduction
This project is my submission for Study Case 1 on **[Google Data Analytics Professional Certificate](https://www.coursera.org/professional-certificates/google-data-analytics) - Capstone Project**.
## Scenario
You are a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives must approve your recommendations, so they must be backed up with compelling data insights and professional data visualizations.

## Data Analysis Process
In order to answer the key business questions, I will follow the steps of the data analysis process: _ask, prepare, process, analyze, share, and act_.

### Phase 1 - Ask
Based on above scenario, I started this project by asking the stakeholder about the project goals. This phase will give me clear understanding about the business tasks, and stay focus on the project goals when applying next step on every phase of data analysis process. This phase will also help me to decide: what data should I get, and how to perform analysis on this kind of data.

Three questions arose after applied this phase:
1. How do annual members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?

### Phase 2 - Prepare
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

| Field name		      | Type      |
| ------------------- | --------- |
| ride_id		          | STRING	  |
| rideable_type		    | STRING	  |
| started_at		      | TIMESTAMP	|
| ended_at		        | TIMESTAMP	|
| start_station_name	| STRING	  |
| start_station_id		| STRING	  |
| end_station_name		| STRING	  |
| end_station_id		  | STRING	  |
| start_lat		        | FLOAT	    |
| start_lng		        | FLOAT	    |
| end_lat		          | FLOAT	    |
| end_lng		          | FLOAT	    |  
| member_casual		    | STRING	  |

Identify total records in `bike_trip_2022` for data cleaning:

```sql
SELECT COUNT(*) AS total_records
FROM bike_trip_2022
```

| total_records |
| ------- |
| 5667717 |

### Phase 3 - Process
To easily identify total ride length and the day of the week, I do the following:
-  I created a column called `ride_length` to calculate the length of each ride by subtracting the column `started_at` from the column `ended_at`
-  Then, I created query using `CASE` to identify the day of the week on each trip, by extracting the date part from column `started_at`

```sql
SELECT
  ride_id,
  rideable_type,
  started_at,
  ended_at,
  (
  SELECT
    TIME_ADD( TIME '0:0:0', INTERVAL TIMESTAMP_DIFF(ended_at, started_at, SECOND) SECOND )) AS ride_length,
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
  member_casual
FROM
  `utopian-saga-394613.cyclistic_data.bike_trip_2022`;
```

_will be updated_
