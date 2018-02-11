    library(knitr)

    ## Warning: package 'knitr' was built under R version 3.4.3

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.4.3

    knitr::opts_chunk$set(echo = TRUE)

Reproducable Research - Week 2 - Project RepData\_PeerAssessment1
=================================================================

Loading and preprocessing the data
----------------------------------

### Show any code that is needed to

1.  Load the data (i.e. read.csv())

<!-- -->

    activity <- read.csv("activity.csv")

1.  Process/transform the data (if necessary) into a format suitable for
    your analysis

<!-- -->

    act_date <- aggregate(activity$steps, by=list(activity$date), FUN=sum, na.rm=TRUE)

What is mean total number of steps taken per day?
-------------------------------------------------

### For this part of the assignment, you can ignore the missing values in the dataset.

1.  Make a histogram of the total number of steps taken each day

<!-- -->

    qplot(act_date$x, 
          binwidth = 5000, geom = "histogram",
          xlab = "Steps", ylab = "Frequency",
          color=I("black"), fill=I("light blue"))

![](PA1_template_files/figure-markdown_strict/step_total_plot-1.png)

1.  Calculate and report the mean and median total number of steps taken
    per day

<!-- -->

    mean(act_date$x)

    ## [1] 9354.23

    median(act_date$x)

    ## [1] 10395

What is the average daily activity pattern?
-------------------------------------------

1.  Make a time series plot (i.e. type = "l") of the 5-minute interval
    (x-axis) and the average number of steps taken, averaged across all
    days (y-axis)

<!-- -->

    act_interval <- aggregate(steps~interval, 
                   FUN=mean, data=activity, na.rm=TRUE)
    plot(act_interval, type="l")

![](PA1_template_files/figure-markdown_strict/interval_plot-1.png)

1.  Which 5-minute interval, on average across all the days in the
    dataset, contains the maximum number of steps?

<!-- -->

    act_interval[which.max(act_interval$steps), ] $interval

    ## [1] 835

Inputting missing values
------------------------

### Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1.  Calculate and report the total number of missing values in the
    dataset (i.e. the total number of rows with NAs)

<!-- -->

    sum(is.na(activity$steps))

    ## [1] 2304

1.  Devise a strategy for filling in all of the missing values in the
    dataset. The strategy does not need to be sophisticated. For
    example, you could use the mean/median for that day, or the mean for
    that 5-minute interval, etc.

*The mean for each interval will be calculated, and each NA will be
replaced with the mean for that interval.*

1.  Create a new dataset that is equal to the original dataset but with
    the missing data filled in.

<!-- -->

    act_date_mean <- aggregate(steps~interval, FUN=mean, data=activity, na.rm=TRUE)
    activity_1 <- activity
    activity_1$steps <-ifelse(is.na(activity_1$steps) == TRUE,
              act_date_mean$steps[act_date_mean$interval %in% activity_1$interval],
              activity_1$steps)

1.  Make a histogram of the total number of steps taken each day and
    Calculate and report the mean and median total number of steps taken
    per day. Do these values differ from the estimates from the first
    part of the assignment? What is the impact of imputing missing data
    on the estimates of the total daily number of steps?

<!-- -->

    act_date2 <- aggregate(steps~date, FUN=sum, data=activity_1, na.rm=TRUE)
    qplot(act_date$x, 
         binwidth = 5000, geom = "histogram",
         xlab = "Steps", ylab = "Frequency",
         color=I("black"), fill=I("yellow"),
         main = "Replaced Step NA values") + 
         theme(plot.title = element_text(hjust=0.5))

![](PA1_template_files/figure-markdown_strict/rev_step_plot-1.png)

    mean(act_date2$steps)

    ## [1] 10766.19

    median(act_date2$steps)

    ## [1] 10766.19

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

### For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1.  Create a new factor variable in the dataset with two levels --
    "weekday" and "weekend" indicating whether a given date is a weekday
    or weekend day.

<!-- -->

    activity$day <- weekdays(as.Date(activity$date))
    activity$week <- ifelse(activity$day == "Saturday" | activity$day == "Sunday",
         "weekend", "weekday")

1.  Make a panel plot containing a time series plot (i.e. type = "l") of
    the 5-minute interval (x-axis) and the average number of steps
    taken, averaged across all weekday days or weekend days (y-axis).
    The plot should look something like the following, which was created
    using simulated data:

<!-- -->

    act_weekday <- subset(activity, week=="weekday")
    act_weekend <- subset(activity, week=="weekend")
    act_wkday_mean <- aggregate(steps~interval, FUN = mean, data=act_weekday, na.rm=TRUE)
    act_wkend_mean <- aggregate(steps~interval, FUN = mean, data=act_weekend, na.rm=TRUE)
    act_wkday_mean$day <-"weekday"
    act_wkend_mean$day <-"weekend"
    act_wk <- rbind(act_wkday_mean, act_wkend_mean)
    qplot(interval, steps, data=act_wk, geom = "line", facets=day~.)

![](PA1_template_files/figure-markdown_strict/wkday_wkend-1.png)
