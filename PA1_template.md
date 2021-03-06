# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data


```r
d = read.csv(unz("activity.zip", "activity.csv"))
```


## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
## 
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
steps_per_day_list <- d %>% group_by(date) %>% summarise(sum_steps = sum(steps))
steps_per_day_list %>% summarize(mean_steps = mean(sum_steps, na.rm=TRUE))
```

```
## Source: local data frame [1 x 1]
## 
##   mean_steps
## 1   10766.19
```

```r
steps_per_day <- unlist(lapply(steps_per_day_list[!is.na(steps_per_day_list["sum_steps"]),"sum_steps"],as.numeric))
```

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day


```r
hist(steps_per_day)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

3. Calculate and report the mean and median of the total number of steps taken per day


```r
steps_per_day_list %>% summarise(mean_steps = mean(sum_steps, na.rm=TRUE))
```

```
## Source: local data frame [1 x 1]
## 
##   mean_steps
## 1   10766.19
```

```r
steps_per_day_list %>% summarise(median_steps = median(sum_steps, na.rm=TRUE))
```

```
## Source: local data frame [1 x 1]
## 
##   median_steps
## 1        10765
```

## What is the average daily activity pattern?

    Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
activity_list <- d  %>% filter(!is.na(steps)) %>% group_by(interval) %>% summarize ( steps = mean(steps))
plot(activity_list, type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

    Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
 activity_list %>% filter (steps == as.numeric(activity_list %>% summarize(max = max(steps))))
```

```
## Source: local data frame [1 x 2]
## 
##   interval    steps
## 1      835 206.1698
```

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

    Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
d %>% group_by() %>% filter(is.na(steps)) %>% summarize(missing = length(steps))
```

```
## Source: local data frame [1 x 1]
## 
##   missing
## 1    2304
```

```r
d %>% group_by() %>% filter(!is.na(steps)) %>% summarize(present = length(steps))
```

```
## Source: local data frame [1 x 1]
## 
##   present
## 1   15264
```

    Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
    
    My strategy is to use the mean interval steps value for missing intervals


    Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
lookup_interval <- function(inter) { x <- activity_list %>% filter(interval == inter); unlist(x["steps"]) }
d2 <- d
d2$interval_mean_steps <- apply(d2["interval"], 1, function(x) lookup_interval(x["interval"]))
d2$steps <- with(d2, ifelse(is.na(steps),interval_mean_steps,steps))
```

    Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?



```r
steps_per_day_list2 <- d2 %>% group_by(date) %>% summarise(sum_steps = sum(steps))
steps_per_day_list2 %>% summarize(mean_steps = mean(sum_steps))
```

```
## Source: local data frame [1 x 1]
## 
##   mean_steps
## 1   10766.19
```

```r
steps_per_day2 <- unlist(lapply(steps_per_day_list2["sum_steps"],as.numeric))
hist(steps_per_day2)
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

This did not change the data as much as I would have expected.  I added 8 values at the mean of 10766.18, increasting the number of observations in the middle column of the historgram.

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

    Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.



```r
library(lubridate)
d2$day <- wday(d2$date, label = T)
d2$weekend <- factor(with(d2, ifelse(charmatch(day,c("Sat","Sun"), nomatch=FALSE), "Weekend", "Weekday")))
```


    Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
activity_list_weekend <- d2  %>% group_by(weekend, interval) %>% summarize ( steps = mean(steps))
library(lattice)
xyplot(steps~interval | weekend, data = activity_list_weekend,
        type = 'l',
        xlab = 'Interval',
        ylab = 'Number of Steps',
        layout = c(1,2))
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 
