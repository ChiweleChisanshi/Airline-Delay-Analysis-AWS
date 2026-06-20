
# Airline Delays Analysis

## Overview

This project analyzes airline delay data using AWS S3, Amazon Athena, SQL, and Amazon QuickSight. The goal was to explore flight delay patterns, identify the airlines and airports with the most delays, examine common delay causes, and create visualizations to support the analysis.

The project focused on understanding arrival delays, delay causes, airport-level delay patterns, weather impact, and year-over-year delay trends.

## Business Problem

Flight delays affect airlines, airports, passengers, and operational planning. Understanding which airlines, airports, and delay causes contribute most to delays can help improve scheduling, resource planning, and customer experience.

This project explored questions such as:

- Which airlines experience the most frequent delays?
- Which airlines have the longest average delays?
- Which airports have the highest average delay times?
- What are the most common causes of delays?
- How does weather affect flight delays?
- Can delay trends be forecasted over time?

## Tools Used

- AWS S3
- Amazon Athena
- SQL
- Amazon QuickSight
- CSV dataset
- Data visualization
- Exploratory data analysis

## Data Storage and Setup

The dataset was stored in an AWS S3 bucket. I then created a database and external table in Amazon Athena to query the data directly from S3.

The table included fields such as:

- Year
- Month
- Carrier
- Carrier name
- Airport
- Airport name
- Number of arriving flights
- Number of flights delayed more than 15 minutes
- Carrier delay count
- Weather delay count
- National Aviation System delay count
- Security delay count
- Late aircraft delay count
- Cancelled flights
- Diverted flights
- Total arrival delay
- Carrier delay minutes
- Weather delay minutes
- NAS delay minutes
- Security delay minutes
- Late aircraft delay minutes

## Example Athena Table Creation

```sql
CREATE DATABASE airlinedelay;

CREATE EXTERNAL TABLE airlinedelay (
    year INT,
    month INT,
    carrier STRING,
    carrier_name STRING,
    airport STRING,
    airport_name STRING,
    arr_flights FLOAT,
    arr_del15 FLOAT,
    carrier_ct FLOAT,
    weather_ct FLOAT,
    nas_ct FLOAT,
    security_ct FLOAT,
    late_aircraft_ct FLOAT,
    arr_cancelled FLOAT,
    arr_diverted FLOAT,
    arr_delay FLOAT,
    carrier_delay FLOAT,
    weather_delay FLOAT,
    nas_delay FLOAT,
    security_delay FLOAT,
    late_aircraft_delay FLOAT
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
    'serialization.format' = ',',
    'field.delim' = ','
)
LOCATION 's3://chisanshiairlinedelay/'
TBLPROPERTIES ('has_encrypted_data'='false');
````

## Exploratory Data Analysis

The first step was to understand the structure of the dataset using the `DESCRIBE` command in Athena.

```sql
DESCRIBE airlinedelay;
```

I also checked for missing values in important delay-related columns, including:

* arr_delay
* carrier_delay
* weather_delay
* nas_delay
* security_delay
* late_aircraft_delay

The missing percentage was very low, about 0.27%, so I did not drop the rows because the missing values were not large enough to significantly affect the overall analysis.

## Key Analysis Questions

### 1. Airline Performance

This part of the analysis looked at which airlines experienced the most frequent delays and which airlines had the longest average delays.

Example query:

```sql
SELECT
    carrier_name,
    COUNT(*) AS total_delays
FROM airlinedelay
WHERE arr_del15 > 0
GROUP BY carrier_name
ORDER BY total_delays DESC
LIMIT 10;
```

Key findings:

* SkyWest Airlines Inc. had the most frequent delays, with 479 flights delayed more than 15 minutes.
* Envoy Air and Delta Air Lines Inc. followed with 285 and 252 delays.
* American Airlines Inc. had the highest average delay in the Athena query results.
* In the QuickSight visualization, Southwest Airlines showed the longest average delay, followed by JetBlue Airways, American Airlines, and United Airlines.

### 2. Time-Based Trends

This analysis looked at whether delays varied by season, month, day of the week, or time of day.

However, the dataset only contained data from December, so it was not possible to fully analyze seasonal or monthly patterns.

Example query:

```sql
SELECT
    month,
    COUNT(*) AS total_delays,
    AVG(arr_delay) AS avg_delay
FROM airlinedelay
WHERE arr_del15 > 0
GROUP BY month
ORDER BY month;
```

Key finding:

* Since the dataset only covered December, the analysis could not show how delays vary across different months or seasons.

### 3. Airport Impact

This part of the analysis looked at which airports had the highest average delay times.

Example query:

```sql
SELECT
    airport_name,
    AVG(arr_delay) AS avg_delay
FROM airlinedelay
WHERE arr_delay > 0
GROUP BY airport_name
ORDER BY avg_delay DESC
LIMIT 10;
```

Key findings:

* Aspen had the highest average delay in the Athena query results.
* Sun Valley/Hailey/Ketchum followed with the second-highest average delay.
* In the QuickSight visualization, major airports such as San Francisco International, Houston George Bush Intercontinental, Boston Logan International, Tampa International, and Oakland International appeared among airports with high average arrival delays.
* The dataset did not include full origin and destination route information, so route-level delay analysis was limited.

### 4. Delay Causes

This analysis explored the most common causes of flight delays.

Delay categories included:

* Carrier delay
* Weather delay
* National Aviation System delay
* Security delay
* Late aircraft delay

Example query:

```sql
SELECT
    'carrier_delay' AS cause,
    SUM(carrier_delay) AS total_delay_time
FROM airlinedelay

UNION ALL

SELECT
    'weather_delay',
    SUM(weather_delay)
FROM airlinedelay

UNION ALL

SELECT
    'nas_delay',
    SUM(nas_delay)
FROM airlinedelay

UNION ALL

SELECT
    'security_delay',
    SUM(security_delay)
FROM airlinedelay

UNION ALL

SELECT
    'late_aircraft_delay',
    SUM(late_aircraft_delay)
FROM airlinedelay

ORDER BY total_delay_time DESC;
```

Key findings:

* Carrier-related delays were the largest cause of delays, with more than 11.1 million total delay minutes.
* Weather delays were the second-largest cause, with about 3.8 million total delay minutes.
* Security delays and NAS delays also contributed significantly.
* Late aircraft delays were the lowest cause in the general delay cause query.

### 5. Weather Influence

This analysis examined how weather-related delays affected total arrival delays.

Example query:

```sql
SELECT
    weather_ct,
    COUNT(*) AS total_flights,
    AVG(arr_delay) AS avg_delay
FROM airlinedelay
GROUP BY weather_ct
ORDER BY weather_ct;
```

Key findings:

* Weather contributes to flight delays, but it is not the only factor.
* When weather-related delay counts were low, average delays were generally lower.
* As weather-related delay counts increased, delay times became more spread out.
* The QuickSight scatter plot showed that weather-related delays increase variability, but there was no strong linear correlation.

### 6. Forecasting Trends

This analysis looked at whether delay duration changed over time.

Example query:

```sql
SELECT
    year,
    month,
    AVG(arr_delay) AS avg_delay
FROM airlinedelay
WHERE arr_delay > 0
GROUP BY year, month
ORDER BY year, month;
```

Key findings:

* The dataset only contained December data from 2019 and 2020.
* Average delays decreased from 2019 to 2020.
* This decrease may be related to reduced air traffic during the COVID-19 pandemic.
* Because the dataset only covered two December periods, it was not enough to build a strong forecasting model.

## QuickSight Visualizations

I used Amazon QuickSight to create visualizations for:

* Airlines with the most frequent delays
* Airlines with the longest average delays
* Average delay by month
* Average delay by airport
* Delay causes by airline
* Delay causes by airport
* Weather impact on arrival delay
* Average delay by year

These visualizations helped make the patterns easier to understand compared to only using SQL query outputs.

## Project Challenges

Some challenges I experienced during the project included:

* The dataset only contained December data, which limited seasonal analysis.
* The dataset did not include full origin and destination information, which limited route analysis.
* I encountered a permissions error when trying to query Athena data directly in QuickSight.
* As a workaround, I uploaded the CSV file into QuickSight and created visualizations from there.
* Some missing value calculations were initially incorrect and needed to be reviewed.
* The dataset limited forecasting because there were only two years of December data.

## What I Would Improve

If I were to improve this project, I would:

* Use a dataset with more months and years of data
* Include route-level information with origin and destination airports
* Create delay categories such as short, medium, and long delays
* Clean and transform the data before visualization
* Resolve Athena-to-QuickSight permission issues
* Use Amazon SageMaker or another machine learning tool to predict future delays
* Build a more complete dashboard for airline and airport performance

## Skills Demonstrated

* Cloud data storage using AWS S3
* SQL querying with Amazon Athena
* External table creation
* Exploratory data analysis
* Missing value analysis
* Data visualization with Amazon QuickSight
* Delay cause analysis
* Airport and airline performance analysis
* Dashboard interpretation
* Data storytelling
* Troubleshooting cloud permissions
* Analytical thinking


## Conclusion

This project helped me understand how cloud tools can be used to store, query, analyze, and visualize large datasets. I learned how to use AWS S3 and Athena for data storage and querying, and QuickSight for dashboard-style analysis.

The analysis showed that carrier-related delays were the largest contributor to total delay time, weather also played an important role, and larger airports and airlines tended to experience more delays. The project also showed the importance of having complete and well-structured data, especially when trying to analyze trends or forecast future outcomes.

```
```
