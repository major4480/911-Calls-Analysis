# 911-Calls-Analysis
### Objective
Analysis of 911, a log of 911 (emergency) call records from Montgomery County, PA, covering December 10, 2015 – August 24, 2016. I Performed Exploratory data analysis using Python to check the reason why people call 911 the most. The reasons are: EMS(Emergency Medical Service), Fire and Traffic.

## Dataset at a glance

| | |
|---|---|
| Rows | 99,492 calls |
| Columns | `lat`, `lng`, `desc`, `zip`, `title`, `timeStamp`, `twp`, `addr`, `e` |
| Date range | 2015-12-10 to 2016-08-24 (~8.5 months, **no Sep–Nov 2015 or 2016 data**) |
| Missing values | `zip`: 12,855 missing · `twp`: 43 missing · `addr`: 519 missing |
| Unique call titles | 110 |


### Process
-I imported the csv file into the Jupyter Notebook
-In the dataset I had latitude, longitude, address, zip code, title, TimeStamp and twp
-I was able to see that there are 110 unique title codes
-I used seaborn to create a countplot and discovered that the REASON why people call 911 is because of EMS
-I used the .apply() method to create new columns called: Hour, Month and Days of Week.
-I then used seaborn to draw countplot of separate plots, Month and Days of week to check again and still discovered that EMS is the top one.
-
