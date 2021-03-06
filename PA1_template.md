# Reproducible Research: Peer Assessment 1   


## Loading and preprocessing the data


```r
unzip("activity.zip")
activity <- read.csv("activity.csv")
```


## What is the mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

* Calculate the total number of steps taken per day
* Make a histogram of the total number of steps taken each day


```r
steps.bydate <- aggregate(steps ~ date, data=activity, FUN=sum)
hist(steps.bydate$steps, main="Distribution of total steps per day", xlab="Number of steps per day", ylab="Frequency")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 


* Calculate and report the mean and median of the total number of steps taken per day


```r
mean(steps.bydate$steps)
```

```
## [1] 10766.19
```

```r
median(steps.bydate$steps)
```

```
## [1] 10765
```


## What is the average daily activity pattern?

* Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
steps.byinterval <- aggregate(steps ~ interval, data=activity, FUN=mean)
plot(steps.byinterval, type="l")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 


* Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
steps.byinterval$interval[which.max(steps.byinterval$steps)]
```

```
## [1] 835
```


## Imputing missing values

* Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)   


```r
sum(is.na(activity))
```

```
## [1] 2304
```


* Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.   

For the missing values I will use as filler values the means for the 5-minute intervals.


* Create a new dataset that is equal to the original dataset but with the missing data filled in.   


```r
activity_intervals <- merge(activity, steps.byinterval, by="interval", suffixes=c("",".y"))
nas <- is.na(activity_intervals$steps)
activity_intervals$steps[nas] <- activity_intervals$steps.y[nas]
activity <- activity_intervals[,c(1:3)]
```


* Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
steps.bydate <- aggregate(steps ~ date, data=activity, FUN=sum)
hist(steps.bydate$steps, main="Distribution of total steps per day", xlab="Number of steps per day", ylab="Frequency") 
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 

```r
mean(steps.bydate$steps)
```

```
## [1] 10766.19
```

```r
median(steps.bydate$steps)
```

```
## [1] 10766.19
```

The effect of the missing data it seems to not affect the estimates of the total daily number of steps.


## Are there differences in activity patterns between weekdays and weekends?

* Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
daygroup <- function(date) {
    if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
        "weekend"
    } else {
        "weekday"
    }
}
activity$daygroup <- as.factor(sapply(activity$date, daygroup))
```


* Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
library(plyr)
library(lattice)

activity <- ddply(activity, .(interval, daygroup), summarise, steps = mean(steps))

xyplot(steps ~ interval | daygroup, data = activity,
       layout = c(1, 2), type = "l", 
       xlab="Interval", ylab="Number of steps")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 



