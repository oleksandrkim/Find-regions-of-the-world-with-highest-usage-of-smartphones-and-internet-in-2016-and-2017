# Find regions of the world with the highest usage of smartphones and social media in 2016 and 2017 with Hive
Aggregating, merging datasets, creating partitions in Hive to analyze the state of smartphones and Internet across countries of the world  
Countries available: 39

Plan:
1. Load data in two tables, one holds population data, second holds social media and smartphone data
2. Merge two tables
3. Use CASE to divide data by regions
4. Group by regions
5. Create partitions

Population data was taken from [World Bank](https://data.worldbank.org/indicator/SP.POP.TOTL)
Data about smartphone and social media usage was taken from [Pew Research Center](http://www.pewglobal.org/2018/06/19/social-media-use-continues-to-rise-in-developing-countries-but-plateaus-across-developed-ones/)

**Subset of population data**


| country|year|population|
| -------|----|----------|
|Aruba|2016|104822|
|Aruba|2017|105264|
|Afghanistan|2016|34656032||
|Afghanistan|2017|35530081|


**Subset of internet/smartphone data**


|country|internet_use|smartphone_own|social_media_usage|year|
| ----|---|----|-----|-----|
|United States|0.89|0.77|0.69|2017|
|Canada|0.91|0.71|0.68|2017|
|France|87|0.62|0.53|2017|
|Germany|0.87|0.72|0.4|2017|


**Create a database**

```
CREATE DATABASE project;
USE project;
```

**Create and load data insde a table that holds smartphone/internet data**
```
CREATE EXTERNAL TABLE IF NOT EXISTS internet_data (
country STRING,
internet_use FLOAT,
smartphone_own FLOAT,
social_media_usage FLOAT,
year INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LOCATION '/user/hirwuser864/internet_data'
TBLPROPERTIES ('creator'='me', 'created_on' = '2018-08-10',
  'description'='This table holds internet data', "skip.header.line.count"="1");

SELECT * FROM internet_data
LIMIT 10;
```
**Result of a select statement**
>United States   0.89    0.77    0.69    2017 <br>
>Canada  0.91    0.71    0.68    2017 <br>
>France  87.0    0.62    0.53    2017 <br>
>Germany 0.87    0.72    0.4     2017 <br>
>Greece  0.66    0.53    0.45    2017 <br>
>Hungary 0.74    0.61    0.56    2017 <br>
>Italy   0.71    0.67    0.48    2017 <br>
>Netherlands     0.93    0.8     0.61    2017 <br>
>Poland  0.75    0.57    0.46    2017 <br>
>Spain   0.87    0.79    0.59    2017 <br>

**Create and load data insde a table that holds population data**
```
CREATE EXTERNAL TABLE IF NOT EXISTS population_data (
country STRING,
year INT,
Population INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LOCATION '/user/hirwuser864/population_data'
TBLPROPERTIES ('creator'='me', 'created_on' = '2018-08-10',
  'description'='This table holds population data', "skip.header.line.count"="1");

SELECT * FROM population_data
LIMIT 10;
```

**Result of a select statement**

>Aruba   2016    104822 <br>
>Aruba   2017    105264 <br>
>Afghanistan     2016    34656032 <br>
>Afghanistan     2017    35530081 <br>
>Angola  2016    28813463 <br>
>Angola  2017    29784193 <br>
>Albania 2016    2876101 <br>
>Albania 2017    2873457 <br>
>Andorra 2016    77281 <br>
>Andorra 2017    76965 <br>

**Merge these two tables on country and year**
```
CREATE TABLE merged_i_p as 
SELECT i.country, i.year, i.internet_use, i.smartphone_own, i.social_media_usage, p.population
FROM internet_data i INNER JOIN population_data p
ON i.country = p.country AND i.year = p.year;

SELECT * FROM merged_i_p
LIMIT 10;
```

>Argentina       2016    0.71    0.48    0.59    43847430 <br>
>Argentina       2017    0.78    0.65    0.65    44271041 <br>
>Australia       2016    0.93    0.79    0.7     24210809 <br>
>Australia       2017    0.93    0.82    0.69    24598933 <br>
>Brazil  2016    0.6     0.41    0.48    207652865 <br>
>Brazil  2017    0.7     0.54    0.53    209288278 <br>
>Canada  2016    0.91    0.72    0.65    36264604 <br>
>Canada  2017    0.91    0.71    0.68    36708083 <br>
>Chile   2016    0.78    0.65    0.66    17909754 <br>
>Chile   2017    0.78    0.72    0.63    18054726 <br>

**Add region to a table**  
6 regions is total: Europe, Asia, Africa, North America, Latin America and Middle East

```
CREATE TABLE merge_continent as 
SELECT *,
CASE
WHEN country in ( 'United States', 'Canada')  THEN 'North America'
WHEN country in ( 'France', 'Germany', 'Greece', 'Hungary', 
  'Italy', 'Netherlands', 'Poland', 'Spain', 'Sweden', 
  'United Kingdom', 'Russia')  THEN 'Europe'
WHEN country in ('Australia', 'China', 'India', 'Indonesia', 
  'Japan', 'Philippines', 'South Korea', 'Vietnam')  THEN 'Asia'
WHEN country in ('Israel', 'Jordan', 'Lebanon', 'Tunisia', 'Turkey')  THEN 'Middle East'
WHEN country in ('Ghana', 'Kenya', 'Nigeria', 'Senegal', 
  'South Africa', 'Tanzania')  THEN 'Africa'
WHEN country in ('Argentina', 'Brazil', 'Chile', 'Colombia', 
  'Mexico', 'Peru', 'Venezuela')  THEN 'Latin America'
ELSE null 
END AS continent
FROM merged_i_p;

SELECT * FROM merge_continent
LIMIT 10;
```

>Argentina       2016    0.71    0.48    0.59    43847430        Latin America <br>
>Argentina       2017    0.78    0.65    0.65    44271041        Latin America <br>
>Australia       2016    0.93    0.79    0.7     24210809        Asia <br>
>Australia       2017    0.93    0.82    0.69    24598933        Asia <br>
>Brazil  2016    0.6     0.41    0.48    207652865       Latin America <br>
>Brazil  2017    0.7     0.54    0.53    209288278       Latin America <br>
>Canada  2016    0.91    0.72    0.65    36264604        North America <br>
>Canada  2017    0.91    0.71    0.68    36708083        North America <br>
>Chile   2016    0.78    0.65    0.66    17909754        Latin America <br>
>Chile   2017    0.78    0.72    0.63    18054726        Latin America <br>

## Top-3 continents with highest average rating of smartphone ownership in 2016 and 2017 and region with highest difference (the one that develops the fastest)

```
CREATE TABLE smart_2017 as 
SELECT continent, round(smartphone_own_avg, 3) as rounded_smart FROM continent_group
WHERE year = '2017'
SORT BY rounded_smart DESC;

CREATE TABLE smart_2016 as 
SELECT continent, round(smartphone_own_avg, 3) as rounded_smart FROM continent_group
WHERE year = '2016'
SORT BY rounded_smart DESC;

SELECT continent, rounded_smart FROM smart_2016
SORT BY rounded_smart DESC
LIMIT 3;

SELECT continent, rounded_smart FROM smart_2017
SORT BY rounded_smart DESC
LIMIT 3;
```

**In 2016**

0.72 = 72%

|Continent|% of people who owns smartphones on average|
| ------------- | ------------- |
North America   |0.72|
Europe  |0.626|
|Middle East     |0.496|

**In 2017**

|Continent|% of people who owns smartphones on average|
| ------------- | ------------- |
|North America|   0.74|
|Europe|  0.675|
|Middle East|     0.67|

**Differece between years**

```
CREATE TABLE merged_smart as 
SELECT s.continent, s.rounded_smart as data_2016, d.rounded_smart as data_2017
FROM smart_2016 s INNER JOIN smart_2017 d
ON s.continent = d.continent;

SELECT continent, round(data_2017-data_2016, 3) as diff FROM merged_smart
SORT BY diff DESC;
```

|Continent|% of change from 16 to 17|
| ------------- | ------------- |
|Middle East |    0.174|
|Latin America |  0.117|
|Africa|  0.088|

In summary, the same 3 continents were leaders in number of people who owns smartphones in both 2016 and 2017 - North America, Europa, Middle East. As of continents that develops the fastest, Middle East, Latin America and Africa have the highest development rate.


## Top-3 continents with highest average rating of social media in 2016 and 2017 and region with highest difference (the one that develops the fastest)

```
CREATE TABLE sm_2017 as 
SELECT continent, round(social_media_usage_avg, 3) as rounded_sm FROM continent_group
WHERE year = '2017'
SORT BY rounded_sm DESC;

CREATE TABLE sm_2016 as 
SELECT continent, round(social_media_usage_avg, 3) as rounded_sm FROM continent_group
WHERE year = '2016'
SORT BY rounded_sm DESC;

SELECT continent, rounded_sm FROM sm_2017
SORT BY rounded_sm DESC
LIMIT 3;

SELECT continent, rounded_sm FROM sm_2016
SORT BY rounded_sm DESC
LIMIT 3;
```

**In 2016**

|Continent|% of people who uses social medias on average|
| ------------- | ------------- |
|North America|   0.67|
|Europe | 0.557|
|Middle East  |   0.546|

**In 2017**

|Continent|% of people who uses social medias on average|
| ------------- | ------------- |
|North America  | 0.685|
|Middle East  |   0.632|
|Latin America |  0.581|



**Differece between years**

```
CREATE TABLE merged_sm as 
SELECT s.continent, s.rounded_sm as data_2016, d.rounded_sm as data_2017
FROM sm_2016 s INNER JOIN sm_2017 d
ON s.continent = d.continent;

SELECT continent, round(data_2017-data_2016, 3) as diff FROM merged_sm
SORT BY diff DESC;
```
|Continent|% of change from 16 to 17|
| ------------- | ------------- |
|Middle East|     0.086|
|Africa|  0.065|
|Latin America|   0.061|



In summary, North America and Middle East were leaders in number of people who uses social media in both 2016 and 2017. In contrast, Latin America managed to get 3rd place in 2017, taking a position of Europe which lost its' second place in 2017. As of continents that develops the fastest, Middle East, Latin America and Africa have the fastest spread of social media.

**Create partitions to store data by continents** <br>
Number of continents is not huge, so "dynamic partitioning" can be applied

```
SET hive.exec.dynamic.partition.mode=nonstrict;
INSERT OVERWRITE TABLE merged_partition_dynamic
PARTITION (cont)
SELECT m.*, m.continent
FROM merge_continent m;

SHOW PARTITIONS merged_partition_dynamic;
```

>cont=Africa <br>
>cont=Asia <br>
>cont=Europe <br>
>cont=Latin America <br>
>cont=Middle East <br>
>cont=North America <br>

```
SELECT * FROM merged_partition_dynamic
WHERE cont='Europe'
LIMIT 5;
```

>Germany 2016    0.85    0.66    0.37    82348669        Europe  Europe<br>
>Germany 2017    0.87    0.72    0.4     82695000        Europe  Europe<br>
>Spain   2016    0.9     0.79    0.63    46484062        Europe  Europe<br>
>Spain   2017    0.87    0.79    0.59    46572028        Europe  Europe<br>
>France  2016    0.81    0.58    0.48    66859768        Europe  Europe<br>

