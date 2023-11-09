# fitbit_analysis  
Annette  
2023-10-21  

Here I will show my analysis process working with this Fitbit data set.  

Here is the prompt I was given to work with for this case study:  

> Sršen (CEO, main stakeholder) asks you to analyze smart device usage data in order to gain insight into how consumers use non-Bellabeat smart devices. She then wants you to select one Bellabeat product to apply these insights to in your presentation.  

*Question*: How are people using other smart devices? (In this case, Fitbit devices)    
How can these trends inform the marketing strategy for Bellabeat products?  

### Installing tidyverse package:  

```
library(tidyverse)
```

### Loading dataset:  

```
daily_activity <- readr::read_csv("C:/Users/Annette/OneDrive/Documents/Case Study - Bellabeat/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
```

#### Here are the variables we have to work with:  

```
colnames(daily_activity)
```  
```
##  [1] "Id"                       "ActivityDate"            
##  [3] "TotalSteps"               "TotalDistance"           
##  [5] "TrackerDistance"          "LoggedActivitiesDistance"
##  [7] "VeryActiveDistance"       "ModeratelyActiveDistance"
##  [9] "LightActiveDistance"      "SedentaryActiveDistance" 
## [11] "VeryActiveMinutes"        "FairlyActiveMinutes"     
## [13] "LightlyActiveMinutes"     "SedentaryMinutes"        
## [15] "Calories"
```

Even though the Id column is made up of numbers, each one represents a discrete study participant, so I will cast it as a “character” type.  
The rest are continuous variables - they are numerical and can have an infinite # of values.  

### Preparing the dataset:  

```
daily_activity$ActivityDate <- mdy(daily_activity$ActivityDate)
class(daily_activity$ActivityDate)
```  
```
## [1] "Date"
```
```
daily_activity$Id <- as.character(daily_activity$Id)
class(daily_activity$Id)
```
```
## [1] "character"
```

### Exploratory Data Analysis  

#### What type of variation occurs within my variables?  

> Every variable has its own pattern of variation, and the best way to understand that pattern is to visualise the distribution of the variable’s values.

– [R for Data Science](https://r4ds.had.co.nz/exploratory-data-analysis.html)  

##### Categorical/discrete values - bar chart  
My only categorical value is Id.  

```
ggplot2::ggplot(data = daily_activity) +
  geom_bar(mapping = aes(x = Id)) +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))
```

![download](https://github.com/annettericci/bellabeat_analysis/assets/149432800/45494e41-e737-48a3-a3d9-66f340fc8740)

After changing the ID #’s to character values, I plotted the freqeuency of each participant in the dataset. One participant ID seems to be an outlier and has a much lower number of observations than the others. I don’t think I will exclude them, but I wanted to point it out.  

I wanted to see how many observations there were for this ID, so using the table() function, I found out there are only 4 observations for this participant.  

```
table(daily_activity$Id)
```
```
## 
## 1503960366 1624580081 1644430081 1844505072 1927972279 2022484408 2026352035 
##         31         31         30         31         31         31         31 
## 2320127002 2347167796 2873212765 3372868164 3977333714 4020332650 4057192912 
##         31         18         31         20         30         31          4 
## 4319703577 4388161847 4445114986 4558609924 4702921684 5553957443 5577150313 
##         31         31         31         31         31         31         30 
## 6117666160 6290855005 6775888955 6962181067 7007744171 7086361926 8053475328 
##         28         29         26         31         26         31         31 
## 8253242879 8378563200 8583815059 8792009665 8877689391 
##         19         31         31         29         31
```

##### Continuous values - histogram  

Here I did an inital histogram looking at the frequency of TotalSteps:  

```
ggplot(data = daily_activity) +
  geom_histogram(mapping = aes(x = TotalSteps), binwidth = 500) +
  scale_x_continuous(n.breaks = 15)
```

![download](https://github.com/annettericci/bellabeat_analysis/assets/149432800/7a324b1e-bb15-4cd1-9284-565ca836ba21)  

Here we see that most of the “bell” falls between about ~1500 steps and ~13,500 steps. There are a few noticeable outliers. The most frequent number of TotalSteps is actually 0.  

```
daily_activity %>% dplyr::count(TotalSteps)
```
```
## # A tibble: 842 × 2
##    TotalSteps     n
##         <dbl> <int>
##  1          0    77
##  2          4     1
##  3          8     1
##  4          9     1
##  5         16     1
##  6         17     1
##  7         29     1
##  8         31     1
##  9         42     1
## 10         44     1
## # ℹ 832 more rows
```

It looks like the value 0 for TotalSteps occurs 77 times. This is only about 8% of the total observations, so the histogram may look a bit skewed. I will try again, excluding values of 0, just to get a better look at the other values:  

```
TotalSteps_not0 <- daily_activity %>% filter(daily_activity$TotalSteps != 0)
```
```
ggplot(data = TotalSteps_not0) +
  geom_histogram(mapping = aes(x = TotalSteps), binwidth = 500) +
  scale_x_continuous(n.breaks = 15) +
  theme(axis.title.x = element_text(vjust= -2)) +
  ggtitle("Total Daily Steps (excluding 0)")
```

![download](https://github.com/annettericci/bellabeat_analysis/assets/149432800/677bf433-4996-49df-bb18-18174ede8e92)  

This does provide a more balanced look at where most of the observations fall.  

```
summary(TotalSteps_not0$TotalSteps)
```
```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       4    4923    8053    8319   11092   36019
```

For TotalSteps not equalling zero, the average number is 8,319.  
There are a couple more outliers, such as the max number of 36,019, but since they only occur once I won’t worry about them as much.  


Perhaps we can chalk up the 77 observations of 0 TotalSteps to people just not wanting to or being able to wear their fitness trackers some days, or maybe just forgetting to charge them. Whatever the reason, it’s probably understandable that during the study period of 31 days, you might not be able to wear the device on every single day.  


Most of the observations fall around/slightly below the commonly known “10,000 steps a day” suggestion. But, are these casual walking steps, or vigorous activity steps? Let’s find out:  

##### Creating a data frame for average activity intensity:  

```
mean_intensity <- c(mean(daily_activity$SedentaryMinutes), 
                      mean(daily_activity$LightlyActiveMinutes),
                      mean(daily_activity$FairlyActiveMinutes),
                      mean(daily_activity$VeryActiveMinutes))
```
```
type_intensity <- c("Sedentary", "Lightly Active", "Fairly Active", "Very Active")
```
```
intensity_averages <- data.frame(type_intensity, mean_intensity)
```
```
intensity_averages
```
```
##   type_intensity mean_intensity
## 1      Sedentary      991.21064
## 2 Lightly Active      192.81277
## 3  Fairly Active       13.56489
## 4    Very Active       21.16489
```

We can see that the majority of time spent wearing these fitness trackers is considered sedentary. The average sedentary time is 991 minutes, or about 16 hours. This makes sense since the participants are also wearing these trackers to sleep. Let’s make a new df to account for that:  

###### First, I need to import the sleep data:  

```
sleep_day <- readr::read_csv("C:/Users/Annette/OneDrive/Documents/Case Study - Bellabeat/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
```

###### Previewing the sleep data:  

```
head(sleep_day)
```
```
## # A tibble: 6 × 5
##           Id SleepDay        TotalSleepRecords TotalMinutesAsleep TotalTimeInBed
##        <dbl> <chr>                       <dbl>              <dbl>          <dbl>
## 1 1503960366 4/12/2016 12:0…                 1                327            346
## 2 1503960366 4/13/2016 12:0…                 2                384            407
## 3 1503960366 4/15/2016 12:0…                 1                412            442
## 4 1503960366 4/16/2016 12:0…                 2                340            367
## 5 1503960366 4/17/2016 12:0…                 1                700            712
## 6 1503960366 4/19/2016 12:0…                 1                304            320
```
```
dplyr::n_distinct(sleep_day$Id)
```
```
## [1] 24
```

##### Interestingly, there are only 24 distinct participants in the sleep data set, versus 33 in the daily activity data set.  

```
mean(sleep_day$TotalMinutesAsleep)
```
```
## [1] 419.4673
```

The average for minutes spent sleeping is 419, or about 7 hours for the 24 participants who used the sleep tracking feature.  

###### Making a new data frame after subtracting sleeping minutes:  

```
mean_intensity2 <- c((mean(daily_activity$SedentaryMinutes) - mean(sleep_day$TotalMinutesAsleep)), 
                    mean(daily_activity$LightlyActiveMinutes),
                    mean(daily_activity$FairlyActiveMinutes),
                    mean(daily_activity$VeryActiveMinutes))
```
```
intensity_averages2 <- data.frame(type_intensity, mean_intensity2)
```

I also added a column showing hours as well as minutes.  

```
intensity_averages2$hours <- mean_intensity2/60
```

After making this new table, I decided I still wanted to include time spent sleeping, just in a separate column. So, I made a final data frame by adding one more row with rbind():  

```
activity_intensity <- rbind(intensity_averages2, c("Asleep", mean(sleep_day$TotalMinutesAsleep), mean(sleep_day$TotalMinutesAsleep)/60))
```
```
activity_intensity
```
```
##   type_intensity  mean_intensity2             hours
## 1      Sedentary 571.743325949204  9.52905543248673
## 2 Lightly Active 192.812765957447  3.21354609929078
## 3  Fairly Active 13.5648936170213 0.226081560283688
## 4    Very Active 21.1648936170213 0.352748226950355
## 5         Asleep 419.467312348668   6.9911218724778
```

I still didn’t like how many decimals there were, so I decided to change the numeric values to integers. In trying to do so, I discovered they were actually cast as character types:    

```
class(activity_intensity$mean_intensity2)
```
```
## [1] "character"
```
```
class(activity_intensity$hours)
```
```
## [1] "character"
```

Changing variables to integer types:    

```
activity_intensity$mean_intensity2 <- as.integer(activity_intensity$mean_intensity2)

class(activity_intensity$mean_intensity2)
```  
```
## [1] "integer"
```  
```
activity_intensity$hours <- as.integer(activity_intensity$hours)

class(activity_intensity$hours)
```  
```
## [1] "integer"
```  

And, just to make it look a litter nicer, I changed the column names:  

```
colnames(activity_intensity) <- c("Activity Intensity", "Minutes", "Hours")
```  

###### The final table:  

```
activity_intensity
```  
```
##   Activity Intensity Minutes Hours
## 1          Sedentary     571     9
## 2     Lightly Active     192     3
## 3      Fairly Active      13     0
## 4        Very Active      21     0
## 5             Asleep     419     6
```

So from this new table, we can see that, on average, most time spent wearing the fitbit fitness tracker are considered sedentary, approximately 9 hours (not including sleep), with about 3 hours per day being considered lightly active, with Fairly and Very active both falling under 30 mins each. This could mean people aren’t typically exercising at higher intensities as often, prioritizing lower heart-rate exercises, or perhaps choosing to do higher intensity exercises for shorter periods of time, such as with HIIT (high intensity interval training).  

#### What type of covariation occurs between my variables?  

Now I’m going to explore some potential relationships between the variables.  
First, let’s check out the relationship between TotalSteps and Calories:  

```
ggplot(data = daily_activity, aes(x = TotalSteps, y = Calories)) +
  geom_point() +
  geom_smooth()
```
```
## `geom_smooth()` using method = 'loess' and formula = 'y ~ x'
```

![download](https://github.com/annettericci/bellabeat_analysis/assets/149432800/b68efb18-938b-4032-a36a-9ac28ffd3118)  

Clearly, there is a positive correlation between number of steps taken and calories burned. That’s to be expected, but are participants using their fitness tracker data to lose weight? Let’s check out the weight log:  

###### Importing the weight log data:  

```
weight_log <- readr::read_csv("C:/Users/Annette/OneDrive/Documents/Case Study - Bellabeat/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
```

###### Previewing the data:  

```
head(weight_log)
```
```
## # A tibble: 6 × 8
##           Id Date       WeightKg WeightPounds   Fat   BMI IsManualReport   LogId
##        <dbl> <chr>         <dbl>        <dbl> <dbl> <dbl> <lgl>            <dbl>
## 1 1503960366 5/2/2016 …     52.6         116.    22  22.6 TRUE           1.46e12
## 2 1503960366 5/3/2016 …     52.6         116.    NA  22.6 TRUE           1.46e12
## 3 1927972279 4/13/2016…    134.          294.    NA  47.5 FALSE          1.46e12
## 4 2873212765 4/21/2016…     56.7         125.    NA  21.5 TRUE           1.46e12
## 5 2873212765 5/12/2016…     57.3         126.    NA  21.7 TRUE           1.46e12
## 6 4319703577 4/17/2016…     72.4         160.    25  27.5 TRUE           1.46e12
```
```
dplyr::n_distinct(weight_log$Id)
```
```
## [1] 8
```

Surprisingly, there are only 8 unique participants who used the weight tracking feature. So, there is not enough data to draw any conclusions about the relationship between activity and weight.  

## Conclusions  

- Based on the # of unique participant IDs in various tables, it appears that activity tracking was used by all participants, while sleep tracking was only used by 24, or 72% of participants, and weight tracking was only used by 8, or 24% of participants.  
- The most common activity type tracked was Sedentary, followed by Lightly Active, with Fairly and Very Active falling pretty far behind.

### Recommendations:

- Focus marketing efforts on a more casual user type, perhaps highlighting Bellabeat products’ user-friendliness and showing various activity types in ads, including walking.  
- Collect survey data on the sleep tracking feature to see what could be implemented to make it more useful.  
- Perhaps don’t focus as much on the weight tracking feature since it doesn’t seem to be used very much.  
