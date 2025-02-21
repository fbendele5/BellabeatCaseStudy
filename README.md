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

## Transformation 

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
```
    n_distinct(daily_activity$Id)
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

```
    # Move onto the daily_activity data
    head(daily_activity)
# The LoggedActivitiesDistance and SedentaryActiveDistance columns lack valuable information so it will be removed from the analysis 
daily_activity <- daily_activity[c(-6, -10)]
# Renaming column
colnames(daily_activity)[2] = "Date"
# View updated Data
head(daily_activity)
```

```
    # Finallythe hourly_steps data
head(hourly_steps)

# The time associated with the date is relevant so it must be kept, but it will be easier to work with if it's separated into its own column 
hourly_steps <- hourly_steps %>% separate(ActivityHour, c("Date", "Hour"), sep = "^\\S*\\K")
# View the updated dataframe
head(hourly_steps)

#Because the Id variable is currently numerical but should be used as nominal, the format must be changed in each data set.
daily_activity$Id <- as.character(daily_activity$Id)
daily_sleep$Id <- as.character(daily_sleep$Id)
hourly_steps$Id <- as.character(hourly_steps$Id)
```

## Analysis
1. Graph key variables and look for outliers in the data.

```
    summary(daily_activity$TotalSteps)
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0    3790    7406    7638   10727   36019
```

```
    # Create a box and whisker plot for outliers
    ggplot(daily_activity, aes(x = TotalSteps)) +
  geom_boxplot()

    ## The majority of the daily total steps revolve around 4000-11000, which indicates that there may be possible outliers on the high end.
```

``` steps_upper <- quantile(daily_activity$TotalSteps, .9985, na.rm = TRUE) 
# This shows that 99.85% of the observations are at 28,680 or below. In general, values above this number are more than 3 standard deviations from the mean, which reveals that they are outliers. 
```

2. Extract more information by running descriptive statistics

### Sleep Data

Find the average amount of sleep for each participant

```
mean_sleep <- sleep %>% 
    group_by(Id) %>%
    summarize(mean_sleep = mean(TotalMinutesAsleep)) %>%
    select(Id, mean_sleep) %>%
    arrange(mean_sleep) %>%
    as.data.frame()
    head(mean_sleep)
```

```
##           Id mean_sleep
## 1 2320127002    61.0000
## 2 7007744171    68.5000
## 3 4558609924   127.6000
## 4 3977333714   293.6429
## 5 1644430081   294.0000
## 6 8053475328   297.0000
```
Find the percentage of time the participants actually spent sleeping while laying in bed

```
sleep %>%
    group_by(Id) %>%
    mutate(percent_sleep = (TotalMinutesAsleep/TotalTimeInBed)*100) %>% 
    select(Id, percent_sleep) %>% 
    summarize(avg_persleep = mean(percent_sleep)) %>% 
    arrange(avg_persleep) %>% 
    mutate_if(is.numeric, round, 2)
```

```
## # A tibble: 24 × 2
##   Id         avg_persleep
##   <chr>             <dbl>
## 1 3977333714         63.4
## 2 1844505072         67.8
## 3 1644430081         88.2
## 4 2320127002         88.4
## 5 4558609924         90.7
## 6 2347167796         91.0
## 7 5553957443         91.5
## 8 8378563200         91.9
## 9 4445114986         92.5
## 10 4020332650         93.0
## # ℹ 14 more rows
```
Most participants slept around 90% of the time they spent in bed with only 4 participants spending a smaller percentage of time sleeping with the lowest being 63.37%

### Activity Levels

Summary stats of different activity levels

```
library(psych)

activity_level <- activity[9:12] 
describe(activity_level)
```

```
##                     vars   n   mean     sd median trimmed    mad min
## VeryActiveMinutes       1 938  20.91  32.35      4   13.92   5.93   0
## FairlyActiveMinutes     2 938  13.50  19.94      6    9.32   8.90   0
## LightlyActiveMinutes    3 938 192.58 109.02    199  193.33 102.30   0
## SedentaryMinutes        4 938 991.29 301.57   1059  998.80 388.44   0
##                      max range  skew kurtosis   se
## VeryActiveMinutes     210   210  2.15     5.61 1.06
## FairlyActiveMinutes   143   143  2.49     8.05 0.65
## LightlyActiveMinutes  518   518 -0.04    -0.37 3.56
## SedentaryMinutes     1440  1440 -0.29    -0.68 9.85
```

Activity levels by participant

```
activity_id <- activity %>%
    group_by(Id) %>% 
    summarize(sum_very = sum(VeryActiveMinutes), 
    sum_fairly = sum(FairlyActiveMinutes), 
    sum_lightly = sum(LightlyActiveMinutes), 
    sum_sed = sum(SedentaryMinutes)) %>% 
    select(Id, sum_very, sum_fairly, sum_lightly, sum_sed) %>% 
    as.data.frame()
head(activity_id)
```

```
##           Id sum_very sum_fairly sum_lightly sum_sed
## 1 1503960366     1200        594        6818   26293
## 2 1624580081       83        117        4587   37970
## 3 1644430081      287        641        5354   34856
## 4 1844505072        4         40        3579   37405
## 5 1927972279       41         24        1196   40840
## 6 2022484408     1125        600        7981   34490
```

###Steps

Find which hour of the day had the most steps taken on average

```
steps_hourly %>% 
    group_by(Hour) %>% 
    summarize(mean_steps = mean(StepTotal)) %>% 
    select(Hour, mean_steps) %>% 
    arrange(desc(mean_steps)) %>% 
    head(1)
```

```
# A tibble: 1 × 2
  Hour          mean_steps
  <chr>              <dbl>
1 " 6:00:00 PM"       599.
```
6 PM had the most steps taken with an average of around 600 steps

#Create a data frame with average hourly steps for visualizations later

```
mean_steps <- steps_hourly %>%  
    group_by(Hour) %>% 
    summarize(mean_steps = mean(StepTotal)) %>% 
    select(Hour, mean_steps) %>% 
    arrange(desc(Hour)) %>% 
    as.data.frame() 
```

Find the mean and standard deviation for total steps taken by participant

```
steps_byId <- steps_hourly %>% 
    group_by(Id) %>% 
    summarize(mean_steps_id = mean(StepTotal), sd_steps_id = sd(StepTotal)) %>%
    mutate_if(is.numeric, round, 2) %>% 
    as.data.frame()
head(steps_byId)
```
```
##           Id mean_steps_id sd_steps_id
## 1 1503960366        522.38      836.48
## 2 1624580081        241.51      760.46
## 3 1644430081        307.81      589.61
## 4 1844505072        109.36      232.06
## 5 1927972279         38.59      164.17
## 6 2022484408        477.87      861.30
```

## Visualization

Visualize relationship between steps taken in a day and sedentary minutes
```
ggplot(data=activity, aes(x=TotalSteps, y=SedentaryMinutes)) + 
    geom_point() + 
    geom_smooth() + 
    labs(title="Total Steps Versus Sedentary Minutes", x = "Steps", y = "Minutes")
```
There shows to be no correlation between total daily steps taken and sedentary minutes. This can be confirmed with a simple linear regression:

```
sed_steps_lr <- lm(SedentaryMinutes ~ TotalSteps, activity)
summary(sed_steps_lr)
```

```
##
## Call:
## lm(formula = SedentaryMinutes ~ TotalSteps, data = activity)
## 
## Residuals:
##    Min      1Q  Median      3Q     Max 
## -1145.7  -234.3   103.0   240.8   539.1 
##
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  1.146e+03  1.697e+01   67.53   <2e-16 ***
## TotalSteps  -2.041e-02  1.873e-03  -10.89   <2e-16 ***
## ---
## Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
##
## Residual standard error: 284.2 on 936 degrees of freedom
## Multiple R-squared:  0.1125,	Adjusted R-squared:  0.1116 
## F-statistic: 118.7 on 1 and 936 DF,  p-value: < 2.2e-16
``` 
Results confirm there is little correlation with an r^2 value of .11

Visualize average amount of time participants slept each night during the length of the study
```
options(scipen = 999)
ggplot(mean_sleep, aes(x= Id, y = mean_sleep)) +
    geom_col(aes(reorder(Id, +mean_sleep), y= mean_sleep)) +
    labs(title = "Average Minutes of Sleep", x = "Participant Id", y = "Minutes") + 
    theme(axis.text.x = element_text(angle = 90)) +
    geom_hline(yintercept = mean(mean_sleep$mean_sleep), color = "green")
```
#This graph displays the average sleep of each participant indivdiually as well as how their sleep compares to the overall average across all participants

Visualize average steps per hour

```
ggplot(mean_steps, aes(x = Hour, y = mean_steps)) + 
+ geom_col(aes(reorder(Hour, +mean_steps), mean_steps)) +
+ theme(axis.text.x = element_text(angle = 90)) +
+ labs(title = "Average Steps Taken Per Hour of Day", x = "Hour", y = "Average Steps")
```
From this graph, the most steps were taken in the evening from 5-7 PM while the least amount of steps were in the middle of the night from 12-4 AM

Now, two data sets that were previously created, activity_id and steps_byId, will be combined to find new relationships between key variables

```
combined_data <- merge(activity_id, steps_byId, by = "Id")
#Put the numerical variables into a separate data frame then run a correlation matrix
num_data <- combined_data[-1]
cor(num_data)
```

```
##                 sum_very sum_fairly sum_lightly     sum_sed
## sum_very       1.00000000  0.3310092  0.08229265 -0.01609547
## sum_fairly     0.33100923  1.0000000  0.12218239 -0.14896700
## sum_lightly    0.08229265  0.1221824  1.00000000 -0.02534821
## sum_sed       -0.01609547 -0.1489670 -0.02534821  1.00000000
## mean_steps_id  0.69782565  0.4695951  0.50800458 -0.19685056
## sd_steps_id    0.71214478  0.4478268  0.25023909 -0.04474601
##              mean_steps_id sd_steps_id
## sum_very          0.6978256  0.71214478
## sum_fairly        0.4695951  0.44782678
## sum_lightly       0.5080046  0.25023909
## sum_sed          -0.1968506 -0.04474601
## mean_steps_id     1.0000000  0.91264532
## sd_steps_id       0.9126453  1.00000000
```

# Based on the correlation matrix, there is little correlation between the different activity levels
# There is a moderate correlation(.7) between mean steps taken and very active minutes
```
ggplot(combined_data, aes(x = mean_steps_id, y = sum_very)) + 
+ geom_point() +
+ labs(title = "Average Steps Taken In A Day Compared to Very Active Minutes", x = "Average Steps", y = "Very Active Minutes")
```

There appears to be a moderate upwards trend of "very active minutes" increasing while average steps in a day increases

In addition, I completed additional visualizations in Tablea, which can be viewed here: Tableau Bellabeat Dashboard URL


**SCREENSHOT OF DASHBOARD**

The descriptive  statistical analyses and visualizations reveal the following smart device usage trends:

- Sedentary minutes took up the majority of participants' days and were consistent throughout the week
- Average "very active minutes" were also consistent throughout the week around 20 minutes each day
- Participants on average slept the most on Sundays, which is also the same day that saw the least amount of steps
- Most steps were taken on Tuesdays and Saturdays
- The fewest steps were taken at 3:00 AM and the most steps were taken at 6 PM
- Participants roughly slept around 390 minutes or 6.5 hours per night
- Users that take more steps per day are more likely to engage in "very active minutes"

### Suggestions

Several recommendations based on the analysis and exploration can be created for the smart fitness devices:
 1. A very small amount of customers use the weight log feature so this doesn't appear to be a selling point. Instead, the focus should be on other features, such as sleep or steps tracking. Further research should be considered to look into how the weight log could be more marketable.

2. The data reveals that when active, participants engage the most in "light" activity and didn't contain many "very active" minutes each day. As an incentive, the company could set up a "level up" feature in which participants can earn points based on the amount of time they were active. Higher levls of activtiy would earn more points. Such a feature can motivate users to engage in more active minutes.

3. On Sundays, there is a 1000 step decrease compared to the other days of the week. On this day, a notification can appear in the morning with a goal to hit a certain number of steps alongside a reward for reaching a 1-week streak. This notification can help close the gap with the number of steps on Sundays and motivate users to use the device each day of the week.

4. The most usage in a single hour is at 6 PM so it seems that most users spend the majority of the work hours during the day and get most of their steps in after work. An ad can be targeted towards working adults that focused on easily tracking steps throughout the busy work day. A reminder notification can appear at 12 PM and 8 PM to increase their activity levels during other break times (such as lunch and after dinner).

5. On average, participants got less than the CDC recommended 7 hours of sleep per night. Marketing efforts should focus on the device's sleep tracking feature since participants who don't get the recommended amount of sleep may want to track their sleep patterns. Consider marketing along with a mediation or habit tracker to improve sleep. 




