---
title: "Reproducible Research Project 1"
output: html_document
keep_md: true
--- 

## Introduction


It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement -- a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) 

The variables included in this dataset are:

steps: Number of steps taking in a 5-minute interval</br>
date: The date on which the measurement was taken in YYYY-MM-DD format </br>
interval: Identifier for the 5-minute interval in which measurement was taken </br>
The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset. 

## Loading and preprocessing the data

```r
library(data.table)
library(ggplot2)
library(dplyr)


fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileUrl, destfile = paste0(getwd(), '/repdata%2Fdata%2Factivity.zip'), method = "curl")
unzip("repdata%2Fdata%2Factivity.zip")

```

## Reading csv Data into Data.Table. 
```{r echo=FALSE}
activeData <- data.table::fread(input = "activity.csv")
```

## What is  total number of steps taken per day?
 

```r

Total_Steps <- activeData %>%
  group_by(date) %>%
  dplyr::summarize(steps = sum(steps, na.rm=FALSE))

head(Total_Steps,10)

```
```

## Source: local data frame [61 x 3]
## 
##          date total_steps na
## 1  2012-10-01           0  1
## 2  2012-10-02         126  0
## 3  2012-10-03       11352  0
## 4  2012-10-04       12116  0
## 5  2012-10-05       13294  0
## 6  2012-10-06       15420  0
## 7  2012-10-07       11015  0
## 8  2012-10-08           0  1
## 9  2012-10-09       12811  0
## 10 2012-10-10        9900  0
## ..        ...         ... ..
```

## Histogram of the total number of steps taken each day. 

```r

ggplot(Total_Steps, aes(x = steps)) +
  geom_histogram(fill = "blue", binwidth = 1000) +
  labs(title = "Daily Steps", x = "Steps", y = "Frequency")

```
![plot of chunk daily](figure/unnamed-chunk-4-1.png) 


### Calculate and report the mean and median of the total number of steps taken per day. 

```r

mean_median <-Total_Steps %>%
  dplyr::summarize(steps_mean = mean(steps, na.rm=FALSE), steps_median = median(steps, na.rm=FALSE))

```
    ##    Mean_Steps Median_Steps
    ## 1:   10766.19        10765


### What is the average daily activity pattern?
 
```r

IntervalDT  <- activeData %>%
  group_by(interval) %>%
  dplyr::summarize(steps = mean(steps, na.rm=FALSE))

ggplot(IntervalDT, aes(x =interval , y =steps)) +
  geom_line(color = "blue", size = 1) +
  labs(title = "Avg. Daily Steps", x = "Interval", y = "Avg. Steps per day")

```
![plot of chunk histogram](figure/unnamed-chunk-6-1.png) 


### which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
 
```r
maxIntrval  <- IntervalDT %>%
  filter(steps == max(steps))

maxIntrval
```
    ##max_interval   835
    
### Calculate and report the total number of missing values in the dataset
```{r echo=FALSE}
Null_value <- sum(is.na(activeData))
Null_value
  
```
```
## [1] 2304
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval

<b>I used a strategy for filing in all of the missing values by using ifelse statement , if found null value fill with median of steps and return steps </b>

```r
activeData$steps <- ifelse(is.na(activeData$steps),
                           ave(activeData$steps, FUN = function(x) median(x,na.rm=FALSE)),
                           activeData$steps)
head(activeData,10)
```
```
##   steps       date       interval 
##     0      2012-10-01        0    
##     0      2012-10-01        5    
##     0      2012-10-01       10    
##     0      2012-10-01       15    
##     0      2012-10-01       20    
##     0      2012-10-01       25  
##     0      2012-10-01       30    
##     0      2012-10-01       35    
##     0      2012-10-01       40  
##     0      2012-10-01       45 
```
Make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
# total number of steps taken per day
Total_Steps <- activeData %>%
  group_by(date) %>%
  dplyr::summarize(steps = sum(steps, na.rm=FALSE))
# mean and median total number of steps taken per day

mean_median <-Total_Steps %>%
  dplyr::summarize(steps_mean = mean(steps, na.rm=FALSE), steps_median = median(steps, na.rm=FALSE))

```

Type of Estimate | Mean_Steps | Median_Steps
--- | --- | ---
First Part (with na) | 10765 | 10765
Second Part (fillin in na with median) | 9354.23 | 10395

# Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r

activeData[, date := as.POSIXct(date, format = "%Y-%m-%d")]
activeData[, `Day of Week`:= weekdays(x = date)]
activeData[grepl(pattern = "Monday|Tuesday|Wednesday|Thursday|Friday", x = `Day of Week`), "weekday or weekend"] <- "weekday"
activeData[grepl(pattern = "Saturday|Sunday", x = `Day of Week`), "weekday or weekend"] <- "weekend"
activeData[, `weekday or weekend` := as.factor(`weekday or weekend`)]
head(activeData,10)

```


```r
avarge_steps<- aggregate(steps ~ interval + `weekday or weekend`, data=activeData,mean)
ggplot(avarge_steps, aes(x =interval , y =steps)) +
  geom_line(color = "Blue", size = 1) +
  facet_grid(`weekday or weekend` ~ .) +
  xlab("5-minute interval") + 
  ylab("avarage number of steps")

```
![plot of chunk histogram](figure/unnamed-chunk-12-1.png) 
