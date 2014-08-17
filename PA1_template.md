# Reproducible Research: Peer Assessment 1
by Alexander Mazurenko  
16 Aug 2014

## Loading and preprocessing the data
Here we assume that work directory set contains source *activity.zip* file.
Unzip and load data using `read.csv()` functon:

```r
unzip("activity.zip")
activity <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?
First, let's claculate total number of steps performed each day using `aggregate()` function.
Note, that NA values are ignored.  
Using temporary `buffer` object helps to preserve original `activity` data set.

```r
buffer <- aggregate(steps ~ date, data = activity, sum)
```

**1.** **Histogram** of the total number of steps taken each day looks like this:

```r
hist(buffer$steps, main = "Histogram of Total Number of Steps per Each Day")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

**2.** **Mean** and **median** total number of steps taken per day are:

```r
mean(buffer$steps)
```

```
## [1] 10766
```

```r
median(buffer$steps)
```

```
## [1] 10765
```
## What is the average daily activity pattern?
In order to answer this quuestion, we first need to aggregate original data set 
as the average number of steps per interval.  
Aggregated data are saved to `interval.activity` object. Number of rows should be equal to 
number of 5-minutes intervals in one day (*24 hours \* 60 minutes / 5 minutes = 288*).  
Let's use this idea to make a simple sainity check.

```r
interval.activity <- aggregate(steps ~ interval, data = activity, mean)
nrow(interval.activity) == 24*60/5
```

```
## [1] TRUE
```
**1.** Now let's make a time series plot of the 5-minute interval (x-axis) and 
the average number of steps taken, averaged across all days (y-axis):

```r
plot(interval.activity, type = "l", main = "Average Number of Steps Taken per Interval, Across All Days")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

**2.** On the plot built we can clearly see peak. Let's check which 5-minute interval,
on average across all the days in the dataset, contains the maximum number of steps (i.e. our peak).

```r
peak.interval <- interval.activity$interval[which.max(interval.activity$steps)]
peak.interval
```

```
## [1] 835
```
The peak correspondes to interval 8:35
- 8:40.

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as `NA`).
The presence of missing days may introduce bias into some calculations or summaries of the data.

**1.** Let's calculate number of missing values (`NA`s) in the data set:

```r
sum(is.na(activity))
```

```
## [1] 2304
```
**2, 3.** Now we'll fill in these missing values with *mean number of steps per corresponding day*.

```r
activity.complete <- merge(activity, interval.activity, by = "interval", suffixes = c("", ".mean"))
activity.complete$steps[is.na(activity.complete$steps)] <- activity.complete$steps.mean[is.na(activity.complete$steps)]
```

**4.** Finally, let's make a histogram of the total number of steps taken each day 
using the `activity.complete` data set, calculate and report the mean and median 
total number of steps taken per day, and see if these values differ from the 
estimates from the first part of the assignment?

```r
buffer <- aggregate(steps ~ date, data = activity.complete, sum)
hist(buffer$steps, main = "Histogram of Total Number of Steps per Each Day (Complete)")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 

```r
mean(buffer$steps)
```

```
## [1] 10766
```

```r
median(buffer$steps)
```

```
## [1] 10766
```
We see that imputing missing values in the data set we study doens't have a significat effect.

## Are there differences in activity patterns between weekdays and weekends?

**1.** We create a new factor variable `type` in the dataset with two levels – 
“weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
activity.complete$type <- factor(rep("weekday", nrow(activity.complete)), levels = c("weekday", "weekend"))
activity.complete$type[weekdays(as.POSIXlt(activity.complete$date)) %in% c("Saturday", "Sunday")] <- "weekend"
```
**2.** Then we make a panel plot containing a time series plot of the 5-minute interval (x-axis) 
and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
weekday.activity <- aggregate(steps ~ interval, data = activity.complete, mean, 
                              subset = (activity.complete$type == "weekday"))
weekend.activity <- aggregate(steps ~ interval, data = activity.complete, mean, 
                              subset = (activity.complete$type == "weekend"))

par(mfrow=c(2,1))
plot(weekday.activity, type = "l", main = "Average Weekday Activity")
plot(weekend.activity, type = "l", main = "Average Weekend Activity")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12.png) 

From the plot we can clearly see **more activity during "office hours" during the weekend** compared to the same hours on week days.
