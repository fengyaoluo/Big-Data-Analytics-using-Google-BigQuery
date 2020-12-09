# Project 1: Lyft Bay Wheels Data Analysis


#### Problem Statement

- I am a data scientist at Lyft Bay Wheels (https://www.lyft.com/bikes/bay-wheels), formerly known as Ford GoBike, the
  company running Bay Area Bikeshare. I am trying to increase ridership, and
  you want to offer deals through the mobile app to do so. 
  
- What deals do you offer though? Currently, this company has several options which can change over time.  Please visit the website to see the current offers and other marketing information. Frequent offers include: 
  * Single Ride 
  * Monthly Membership
  * Annual Membership
  * Bike Share for All
  * Access Pass
  * Corporate Membership
  * etc.

- Through this project, I will answer these questions: 

  * What are the 5 most popular trips that you would call "commuter trips"? 
  
  * What are your recommendations for offers (justify based on your findings)?
  
  Answers are saved under Project-1-fengyaoluo.ipynb and please refer more analysis and details there. 

---

## Part 1 - Querying Data with BigQuery

- What's the size of this dataset? (i.e., how many trips)

- Question 1: What's the size of this dataset? (i.e., how many trips)
  * Answer: 983,648 
  * SQL query: 
  
  ```sql
  select count(distinct trip_id) 
  from `bigquery-public-data.san_francisco.bikeshare_trips`
  ```

- What is the earliest start date and time and latest end date and time for a trip?

- Question 2: What is the earliest start date and time and latest end date and time for a trip?
  * Answer:
  the earliest start date and time: 2013-08-29 09:08:00 UTC
  the latest end date and time: 2016-08-31 23:48:00 UTC
  
  * SQL query: 
  
  ```sql
  select min(start_date), max(end_date) 
  from `bigquery-public-data.san_francisco.bikeshare_trips`
  ```

- How many bikes are there?

- Question 3: How many bikes are there?
  * Answer: 700
  * SQL query:
  
  ```sql
  SELECT count(distinct bike_number)
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  ```

### Questions of your own
- Make up 3 questions and answer them using the Bay Area Bike Share Trips Data.  These questions MUST be different than any of the questions and queries you ran above.

- Question 1: What is the frequency of trips in the sense of day of the week?
  * Answer: 
  
  | weekday | Trip_Count |
  |---------|------------|
  | 1       | 58790      |
  | 2       | 171181     |
  | 3       | 184037     |
  | 4       | 180103     |
  | 5       | 175816     |
  | 6       | 153543     |
  | 7       | 60178      |

  * SQL query: 
  
  ```sql
  select extract(dayofweek from DATETIME(start_date,
    "America/Los_Angeles")) as weekday,
  count(trip_id) as Trip_Count,
  from `bigquery-public-data.san_francisco.bikeshare_trips` 
  group by weekday
  order by weekday
  ```

- Question 2: What is the frequency of trips in the sense of hour of the day?
  * Answer:
  
  | hour | Trip_Count |
  |------|------------|
  | 0    | 86967      |
  | 1    | 121939     |
  | 2    | 80311      |
  | 3    | 41826      |
  | 4    | 42292      |
  | 5    | 46003      |
  | 6    | 41901      |
  | 7    | 40761      |
  | 8    | 59969      |
  | 9    | 98465      |
  | 10   | 113412     |
  | 11   | 72444      |
  | 12   | 36311      |
  | 13   | 20567      |
  | 14   | 13798      |
  | 15   | 9123       |
  | 16   | 5291       |
  | 17   | 2520       |
  | 18   | 1392       |
  | 19   | 754        |
  | 20   | 784        |
  | 21   | 2687       |
  | 22   | 9525       |
  | 23   | 34606      |
  
  * SQL query: 
  
  ```sql
  select extract(hour from DATETIME(start_date,
    "America/Los_Angeles")) as hour,
  count(trip_id) as Trip_Count,
  from `bigquery-public-data.san_francisco.bikeshare_trips` 
  group by hour
  order by hour
  ```

- Question 3: What is the frequency of trips (hourly) in weekdays?
  * Answer: 
  
  | hour | Trip_Count |
  |------|------------|
  | 0    | 18054      |
  | 1    | 25640      |
  | 2    | 17935      |
  | 3    | 10757      |
  | 4    | 11589      |
  | 5    | 12238      |
  | 6    | 11591      |
  | 7    | 10938      |
  | 8    | 14611      |
  | 9    | 21887      |
  | 10   | 24577      |
  | 11   | 16232      |
  | 12   | 8544       |
  | 13   | 5040       |
  | 14   | 3533       |
  | 15   | 2231       |
  | 16   | 1293       |
  | 17   | 562        |
  | 18   | 277        |
  | 19   | 156        |
  | 20   | 261        |
  | 21   | 946        |
  | 22   | 3283       |
  | 23   | 12431      |
  
  * SQL query:
  
  ```sql
  select extract(hour from DATETIME(start_date,
    "America/Los_Angeles")) as hour,
  count(trip_id) as Trip_Count,
  from `bigquery-public-data.san_francisco.bikeshare_trips` 
  where extract(dayofweek from DATETIME(start_date,
      "America/Los_Angeles")) in (1,5)
  group by  hour
  order by  hour
  ```

### Bonus activity queries (optional - not graded - just this section is optional, all other sections are required)

The bike share dynamic dataset offers multiple tables that can be joined to learn more interesting facts about the bike share business across all regions. These advanced queries are designed to challenge you to explore the other tables, using only the available metadata to create views that give you a broader understanding of the overall volumes across the regions(each region has multiple stations)

We can create a temporary table or view against the dynamic dataset to join to our static dataset.

Here is some SQL to pull the region_id and station_id from the dynamic dataset.  You can save the results of this query to a temporary table or view.  You can then join the static tables to this table or view to find the region:
```sql
#standardSQL
select distinct region_id, station_id
from `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
```

- Top 5 popular station pairs in each region

```sql
select * from (select concat( start_station_id,"-", end_station_id ), count(concat( start_station_id, "-",end_station_id )) as count, start_region,
row_number() over(partition by start_region order by count(concat( start_station_id, "-",end_station_id )) desc) RowNum
from(

with cte as (select distinct region_id, station_id
from `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`)

select a.*, cte.region_id as end_region from (select trips.*, cte.region_id as start_region from `bigquery-public-data.san_francisco.bikeshare_trips` as trips left join 
cte
on trips.start_station_id = cte.station_id ) as a
left join cte on a.end_station_id = cte.station_id) as b
where start_region =end_region
group by concat( start_station_id,"-", end_station_id ), start_region
--order by count desc
)
where RowNum <6
```


start-end station |count|	start_region|	RowNum
----|----|----|----
50-60|	9150|	3|	1
61-50|	7620|	3|	2
50-61|	6888|	3|	3
60-74|	6874|	3|	4
51-70|	6351|	3|	5
35-35|	1184|	5|	1
46-46|	403|	12|	1
7-7|	204|	12|	2

- Top 3 most popular regions(stations belong within 1 region)

```sql
select count(concat( start_station_id, "-",end_station_id )) as count, start_region
from(

with cte as (select distinct region_id, station_id
from `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`)

select a.*, cte.region_id as end_region from (select trips.*, cte.region_id as start_region from `bigquery-public-data.san_francisco.bikeshare_trips` as trips left join 
cte
on trips.start_station_id = cte.station_id ) as a
left join cte on a.end_station_id = cte.station_id) as b
where start_region =end_region
group by start_region
order by count desc
```
region |count|
----|----
3|	571932
5|	1184
12|	607

- Total trips for each short station name in each region



- What are the top 10 used bikes in each of the top 3 region. these bikes could be in need of more frequent maintenance.


```sql
select * from (select bike_number, count(trip_id), start_region,
row_number() over(partition by start_region order by count(trip_id) desc) RowNum
from(

with cte as (select distinct region_id, station_id
from `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`)

select a.*, cte.region_id as end_region from (select trips.*, cte.region_id as start_region from `bigquery-public-data.san_francisco.bikeshare_trips` as trips left join 
cte
on trips.start_station_id = cte.station_id ) as a
left join cte on a.end_station_id = cte.station_id) as b
where start_region =end_region
group by bike_number, start_region)
--order by count desc)
where RowNum <4
```
bike_number|	trip_count|	start_region|	RowNum
----|----|----|----
389|1732|	3|	1
631|	1721|	3|	2
287|	1698|	3|	3
120|	21|	5|	1
201|	20|	5|	2
638|	18|	5|	3
610|	5|	12|	1
419|	5|	12|	2
53|	4|	12|	3


---

## Part 2 - Querying data from the BigQuery CLI 

### Queries

1. Rerun the first 3 queries from Part 1 using bq command line tool (Paste your bq
   queries and results here, using properly formatted markdown):

  * What's the size of this dataset? (i.e., how many trips)
  
  ```
  bq query --use_legacy_sql=false 'select count(distinct trip_id) 
   from `bigquery-public-data.san_francisco.bikeshare_trips`'
  ```

  * What is the earliest start time and latest end time for a trip?
  
  ```
  bq query --use_legacy_sql=false '
  select min(start_date), max(end_date) 
  from `bigquery-public-data.san_francisco.bikeshare_trips` '
  ```

  * How many bikes are there?
  
  ```
  bq query --use_legacy_sql=false '
  SELECT count(distinct bike_number)
  FROM `bigquery-public-data.san_francisco.bikeshare_trips` '
  ```
  

2. New Query (Run using bq and paste your SQL query and answer the question in a sentence, using properly formatted markdown):

  * How many trips are in the morning vs in the afternoon?
  
  ``` 
  bq query --use_legacy_sql=false 'select count(distinct trip_id),
>     case when extract(hour from DATETIME(start_date,
>     "America/Los_Angeles")) in (0,11) Then "Morning" 
>     when extract(hour from DATETIME(start_date,
>     "America/Los_Angeles")) in (12,18) then "Afternoon"
>     else "Evening" end as parts_of_day
>    from `bigquery-public-data.san_francisco.bikeshare_trips`
>    group by parts_of_day'
  ```
  
| parts_of_day | counts |
|--------------|--------|
| Evening      | 786534 |
| Morning      | 159411 |
| Afternoon    | 37703  |


### Project Questions
Identify the main questions you'll need to answer to make recommendations (list
below, add as many questions as you need).

- Question 1: How many commuter trips in the database? (Commuter trips are defined trips happened in the rush hour during weekdays)

- Question 2: How many users were customers when they were doing commuter trips?

- Question 3: What are the top 5 start stations that customers were doing commuter trips?

- Question 4: What are the top 5 end stations that customers were doing commuter trips?

- ...

- Question n: What are the top 5 stations which zero bike available happened the most times there?

### Answers


- Question 1: How many commuter trips in the database?
  * Answer: 154868
  * SQL query:
  
  ```sql
  select count(*) from (select *,Case 
  when ((extract(hour from DATETIME(start_date,"America/Los_Angeles")) between 7 and 9) or (extract(hour from   DATETIME(start_date,"America/Los_Angeles")) between 16 and 18))
  and extract(dayofweek from  DATETIME(start_date,"America/Los_Angeles")) between 1 and 5 Then "Commuter Trip"
  Else "Non Commuter Trip"
  end as CommuterTrip
  from `bigquery-public-data.san_francisco.bikeshare_trips` )
  where CommuterTrip = "Commuter Trip"
  ```


- Question 2:How many users were customers when they were doing commuter trips?
  * Answer:
  
  | subscriber_type | user_counts |
  |-----------------|-------------|
  | Subscriber      | 130266      |
  | Customer        | 24602       |
  
  * SQL query:
  ```sql
  select subscriber_type, count(*) from (select *,Case 
  when ((extract(hour from DATETIME(start_date, "America/Los_Angeles")) between 7 and 9) or (extract(hour from DATETIME(start_date, "America/Los_Angeles")) between 16 and 18))
  and extract(dayofweek from DATETIME(start_date, "America/Los_Angeles")) between 1 and 5 Then "Commuter Trip"
  Else "Non Commuter Trip"
  end as CommuterTrip
  from `bigquery-public-data.san_francisco.bikeshare_trips` )
  where CommuterTrip = "Commuter Trip"
  group by subscriber_type
  ```

- Question 3:What are the top 5 start stations that customers were doing commuter trips?
  * Answer:
  
  | start_station_name                   | counts |
  |--------------------------------------|--------|
  | Embarcadero at Sansome               | 13934  |
  | Harry Bridges Plaza (Ferry Building) | 12441  |
  | Market at 4th                        | 5952   |
  | Powell Street BART                   | 5214   |
  | Embarcadero at Vallejo               | 4945   |
  
  * SQL query:
  ```sql
  select start_station_name , count(trip_id ) as counts from (  select *,Case 
  when ((extract(hour from DATETIME(start_date, "America/Los_Angeles")) between 7 and 9) or (extract(hour from DATETIME(start_date, "America/Los_Angeles")) between 16 and 18))
  and extract(dayofweek from DATETIME(start_date, "America/Los_Angeles")) between 1 and 5 Then "Commuter Trip"
  Else "Non Commuter Trip"
  end as CommuterTrip
  from `bigquery-public-data.san_francisco.bikeshare_trips` 
  where subscriber_type = 'Customer')
  group by start_station_name 
  order by counts desc
  limit 5
  ```
  
- Question 4:What are the top 5 end stations that customers were doing commuter trips?
  * Answer:
  
  | end_station_name                     | counts |
  |--------------------------------------|--------|
  | Embarcadero at Sansome               | 17921  |
  | Harry Bridges Plaza (Ferry Building) | 11200  |
  | Market at 4th                        | 5759   |
  | Powell Street BART                   | 5225   |
  | Embarcadero at Vallejo               | 5212   |

  * SQL query:
  ```sql
  select end_station_name , count(trip_id ) as counts from (  select *,Case 
  when ((extract(hour from DATETIME(start_date, "America/Los_Angeles")) between 7 and 9) or (extract(hour from DATETIME(start_date, "America/Los_Angeles")) between 16 and 18))
  and extract(dayofweek from DATETIME(start_date, "America/Los_Angeles")) between 1 and 5 Then "Commuter Trip"
  Else "Non Commuter Trip"
  end as CommuterTrip
  from `bigquery-public-data.san_francisco.bikeshare_trips` 
  where subscriber_type = 'Customer')
  group by end_station_name 
  order by counts desc
  limit 5
  ```
- ...

- Question n:What are the top 10 stations which zero bike available happened the most times there?
  * Answer:
  
  | station_id | name                                     | counts | longitude | latitude | landmark      |
  |------------|------------------------------------------|--------|-----------|----------|---------------|
  | 62         | 2nd at Folsom                            | 44844  | -122.396  | 37.7853  | San Francisco |
  | 45         | Commercial at Montgomery                 | 44728  | -122.403  | 37.79423 | San Francisco |
  | 48         | Embarcadero at Vallejo                   | 35903  | -122.399  | 37.79995 | San Francisco |
  | 60         | Embarcadero at Sansome                   | 32980  | -122.403  | 37.80477 | San Francisco |
  | 41         | Clay at Battery                          | 32505  | -122.4    | 37.795   | San Francisco |
  | 70         | San Francisco Caltrain (Townsend at 4th) | 32027  | -122.395  | 37.77662 | San Francisco |
  | 73         | Grant Avenue at Columbus Avenue          | 31733  | -122.406  | 37.7979  | San Francisco |
  | 76         | Market at 4th                            | 30800  | -122.405  | 37.78631 | San Francisco |
  | 63         | Howard at 2nd                            | 27938  | -122.398  | 37.78698 | San Francisco |
  | 82         | Broadway St at Battery St                | 25496  | -122.401  | 37.79854 | San Francisco |
  
  * SQL query:
  ```sql
  select status.station_id, name, count(status.bikes_available ) as counts, station.longitude , station.latitude , station.landmark from `bigquery-public-data.san_francisco.bikeshare_status` as status
  left join `bigquery-public-data.san_francisco.bikeshare_stations` as station
  on status.station_id = station.station_id 
  where bikes_available = 0 
  group by station_id, name, station.longitude , station.latitude , station.landmark 
  order by counts desc
  limit 10
  ```

---

## Part 3 - Employ notebooks to synthesize query project results

Please refer to the Project-1-fengyaoluo.ipynb in the folder.


~:q


