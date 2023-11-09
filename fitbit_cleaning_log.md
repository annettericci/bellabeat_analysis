# fitbit_cleaning_log  
Annette R  
2023-10-18  

Here, I’m showing how I cleaned the table “daily_activity” from the following dataset on Kaggle:  
> Name: FitBit Fitness Tracker Data  
> Author: MÖBIUS  
> [Link](https://www.kaggle.com/datasets/arashnic/fitbit)  

### Loading packages:  

```
library(tidyverse)
```  

### Loading Fitbit dataset:  

```
daily_activity <- readr::read_csv("C:/Users/Annette/OneDrive/Documents/Case Study - Bellabeat/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
```

### Previewing data:  

```
head(daily_activity)
```
```
## # A tibble: 6 × 15
##           Id ActivityDate TotalSteps TotalDistance TrackerDistance
##        <dbl> <chr>             <dbl>         <dbl>           <dbl>
## 1 1503960366 4/12/2016         13162          8.5             8.5 
## 2 1503960366 4/13/2016         10735          6.97            6.97
## 3 1503960366 4/14/2016         10460          6.74            6.74
## 4 1503960366 4/15/2016          9762          6.28            6.28
## 5 1503960366 4/16/2016         12669          8.16            8.16
## 6 1503960366 4/17/2016          9705          6.48            6.48
## # ℹ 10 more variables: LoggedActivitiesDistance <dbl>,
## #   VeryActiveDistance <dbl>, ModeratelyActiveDistance <dbl>,
## #   LightActiveDistance <dbl>, SedentaryActiveDistance <dbl>,
## #   VeryActiveMinutes <dbl>, FairlyActiveMinutes <dbl>,
## #   LightlyActiveMinutes <dbl>, SedentaryMinutes <dbl>, Calories <dbl>
```  

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

```
dplyr::n_distinct(daily_activity$Id)
```
```
## [1] 33
```

There are 33 unique participant ID's.  

### Cleaning the data:  

#### Checking for duplicate rows:  
```
nrow(daily_activity)
```
```
## [1] 940
```
```
nrow(dplyr::distinct(daily_activity))
```
```
## [1] 940
```

Using distinct(daily_activity) returns the same number of rows (940) as the original data frame (daily_activity). Therefore, I will assume there are no duplicate rows in the original data set.  

#### Checking for white spaces:  

```
sum(daily_activity == " ")
```
```
## [1] 0
```

Using the sum() function, I determined there are no white spaces in the data set.  

#### Making sure variable types are cast correctly:  

```
class(daily_activity$ActivityDate)
```
```
## [1] "character"
```

Using the class() function shows that the ActivityDate variable is cast as a character type. Below, I will use mdy() to change it to a date type:  

```
daily_activity$ActivityDate <- lubridate::mdy(daily_activity$ActivityDate)
```

Testing the type of the ActivityDate variable again:  

```
class(daily_activity$ActivityDate)
```
```
## [1] "Date"
```

The ActivityDate column has been changed to a date type.  

## Summary:  

- Imported and previewed the data  
- Checked for duplicate rows  
- Checked for excess white spaces  
- Changed variable “ActivityDate” from a character type to a date type
