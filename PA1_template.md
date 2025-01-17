---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

```r
unzip("activity.zip")
activity <- read.csv("activity.csv")
```

1. Process/transform the data (if necessary) into a format suitable for your analysis

```r
totalSteps<-aggregate(steps ~ date,data = activity,sum,na.rm = TRUE)
```

## What is mean total number of steps taken per day?
1. Make a histogram of the total number of steps taken each day

```r
hist(totalSteps$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

2. Calculate and report the **mean** and **median** total number of
   steps taken per day


```r
mean(totalSteps$steps)
```

```
## [1] 10766.19
```

```r
median(totalSteps$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

* Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
stepsInterval<-aggregate(steps ~ interval,data = activity,mean,na.rm = TRUE)
plot(steps ~ interval,data = stepsInterval,type = "l")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

* Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps? 

```r
stepsInterval[which.max(stepsInterval$steps),]$interval
```

```
## [1] 835
```

## Inputing missing values

* Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

* Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

All missing values were given the mean value. There is a function called  **"intervalToSteps"** that fetches the mean number of steps for the passed 5-minute interval. 

```r
intervalToSteps <- function(interval){
    stepsInterval[stepsInterval$interval == interval,]$steps
}
```

* Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
activityFilled<-activity   
count = 0          
for(i in 1:nrow(activityFilled)){
    if(is.na(activityFilled[i,]$steps)){
        activityFilled[i,]$steps<-intervalToSteps(activityFilled[i,]$interval)
        count = count + 1
    }
}
cat("Total ",count, "NA values were filled in with the mean.")  
```

```
## Total  2304 NA values were filled in with the mean.
```

* Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 

```r
totalSteps2<-aggregate(steps~date,data=activityFilled,sum)
hist(totalSteps2$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
mean(totalSteps2$steps)
```

```
## [1] 10766.19
```

```r
median(totalSteps2$steps)
```

```
## [1] 10766.19
```


* Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

The *mean* value is the same as the previous valus as we replaced missing values with the mean.
The *median* value shows a small difference, but it depends on the position of the missing values.

## Are there differences in activity patterns between weekdays and weekends?

* Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
activityFilled$day = ifelse(as.POSIXlt(as.Date(activityFilled$date))$wday%%6==0,"weekend","weekday")
# Sunday and Saturday are the weekend. Otherwise we are a weekday.
activityFilled$day = factor(activityFilled$day,levels=c("weekday","weekend"))
```


* Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was creating using simulated data:

```r
stepsInterval2 = aggregate(steps~interval+day,activityFilled,mean)
# include the lattice libray for xyplot
library(lattice)
xyplot(steps~interval|factor(day),data=stepsInterval2,aspect=1/2,type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->
