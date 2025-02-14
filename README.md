# BellabeatCaseStudy

## Google Data Analytics Capstone Project

### Background Information 

Bellabeat is a high-tech company that manufactures health-focused smart products and founded by Urška Sršen and Sando Mur. Bellabeat has grown rapidly and quickly positioned itself as a tech-driven wellness company for women.

Bellabeat’s co-founders would like to analyze data from non-Bellabeat fitness devices to see how consumers are using these products. The company hopes to use these insights to help guide new marketing strategies for the company.

In this case study, the following key questions will be answered:
1. What are some trends in smart device usage?
2. How could these trends apply to Bellabeat customers?
3. How can these trends help influence Bellabeat marketing strategy?

Regarding the tools used for this case study, R is used to transform and explore the data then Tableau is used to interactively visualize the data.

### Data Preparation 

The data used for this case study can be accessed through Kaggle in 18 CVS files, which can be found [here](https://www.kaggle.com/datasets/arashnic/fitbit).

Information regarding minute-level output for physical activity, heart rate, steps and sleep monitoring can be found. Altogether, thirty Fitbit users were used in this data collection from a one month period only in 2016. With a sample size of 30, it is important to note that this number is not big enough to be a fair representative of all FitBit users. Demographic information about the users (such as gender and age) was not included, which would have been beneficial for marketing analysis to target specific customers.

The file `dailyActivity_merged.csv` offers a good summary of activity and calories burned while the `sleepDay_merged.csv` file looks into sleep data. Both of these files provide information to analyze participant usage. In addition, the file ~hourlySteps_merged.csv~ looks more closely at the timestamp of steps. Since fitness devices are typically used to track overall health and weight, the file `weightLogInfo_merged` will also be used as it contains weight data.

## Transformation and Analysis

All R code can be viewed [here](URL). 

1. Install and load packages
``` install.packages("tidyverse")
    install.packages("ggplot2")
    install.packages("lubridate")
    install.packages("lm.beta))
```

2. Load CSV files containing the necessary data
``` activity <- read.csv("dailyActivity_merged.csv")
    steps_hourly <- read.csv("hourlySteps_merged.csv")
    daily_sleep <- read.csv("sleepDay_merged.csv")
    weight <- read.csv("weightLogInfo_merged.csv")
```
3. Count number of participants in each data set by focusing on distinct IDs
``` n_distinct(daily_activity$Id)
    ## [1] 33 
```

```
    n_distinct(hourly_steps$Id)
    ## [1] 33
```

```
    n_distinct(daily_sleep$Id)
    ## [1] 24
```

```
    n_distinct(weight$Id)
    ## [1] 8
```
Since a very few participants provided weight information, this will be excluded  from the analysis.

5. Start looking at and cleaning up the data sets
```
# Start with the daily_sleep data
head(daily_sleep)

# The 12:00:00 AM time stamp on each observation is redundant so it should be removed to make the data easier to work with
daily_sleep$SleepDay <- (gsub('12:00:00 AM', '', daily_sleep$SleepDay))
# Rename the column
colnames(daily_sleep)[2] = "Date"
# View updated data
head(daily_sleep)
```

## Visualization


### Suggestions
