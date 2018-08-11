# Find regions of the world with highest usage of smartphones and internet in 2016 and 2017 with Hive
Aggregating, merging datasets, creating partitions in Hive to analyze the state of smartphones and Internet across countries of the world  
Countries available: 39

Some questions to answer are

Population data was taken from [World Bank](https://data.worldbank.org/indicator/SP.POP.TOTL)
Data about smartphone and Internet usage was taken from [Pew Research Center](http://www.pewglobal.org/2018/06/19/social-media-use-continues-to-rise-in-developing-countries-but-plateaus-across-developed-ones/)

**Subset of population data**

```
country,year,population
Aruba,2016,104822
Aruba,2017,105264
Afghanistan,2016,34656032
Afghanistan,2017,35530081
```

**Subset of internet/smartphone data**

```
country,internet_use,smartphone_own,social_media_usage,year
United States,0.89,0.77,0.69,2017
Canada,0.91,0.71,0.68,2017
France,87,0.62,0.53,2017
Germany,0.87,0.72,0.4,2017
```

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
TBLPROPERTIES ('creator'='me', 'created_on' = '2018-08-10', 'description'='This table holds internet data', "skip.header.line.count"="1");
```

