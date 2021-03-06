# Reproducible Research: Peer Assessment 1

&nbsp;

### Global options

We first need to load some of the libraries required for the work and set some global options for the Knitr document.


```r
library(knitr)
library(dplyr)
library(ggplot2)
opts_chunk$set(echo = TRUE)
options(scipen=1, digits=2)
```

&nbsp;


## Loading and preprocessing the data

The data archive `activity.zip` is first unzipped into the current directory and the comma separated variable file is loaded into a data frame using the `read.csv()` command in R. 


```r
activitydata <- read.csv("activity.csv")
head(activitydata)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

We can convert the variable in the `date` column into a R date for easier handling. The `steps` column contains some missing data. We can create a new data frame without the rows that contain missing data.


```r
activitydata$date <- as.Date(activitydata$date, "%Y-%m-%d")
activitynona <- filter(activitydata, !is.na(steps))
```

&nbsp;


## What is mean total number of steps taken per day?

Using the `summarise()` function from the `{dplyr}` package we can create a new data frame containing the total number of steps per day. From the new data frame then we can create a histogram of the distribution of total number of steps taken each day.


```r
dailystepssummary <- summarise(group_by(activitynona, date), dailysteps = sum(steps))
dailyhist <- ggplot(dailystepssummary, aes(x=dailysteps))
dailyhist + 
    geom_histogram(col="black", fill="turquoise2", binwidth = 1000) + 
    labs(title="Histogram of total steps per day", x="Total steps (per day)", y="Number of days")
```

![](PA1_template_files/figure-html/dailysteps-1.png) 


```r
meandailysteps <- mean(dailystepssummary$dailysteps)
mediandailysteps <- median(dailystepssummary$dailysteps)
```

The mean for the total number of steps per day is 10766.19 and the median is
10765.

&nbsp;


## What is the average daily activity pattern?

Using the `summarise()` function again we can also calculate the average steps per 5-min interval across all days and then plot the mean values as a time series of intervals along the day.


```r
intervalstepssummary <- summarise(group_by(activitynona, interval), steps = mean(steps))
maxactivity <- intervalstepssummary[which.max(intervalstepssummary$steps), 1][[1]]
intervalsts <- ggplot(intervalstepssummary, aes(x = interval, y = steps))
intervalsts + 
    geom_line(aes(colour = steps)) + 
    scale_colour_gradient(low="blue", high="red") + 
    labs(title="Average daily activity pattern", x="Interval", y="Number of steps") + 
    geom_vline(xintercept = maxactivity, linetype = "dotted")
```

![](PA1_template_files/figure-html/dailypattern-1.png) 

As seen from the graph and calculated using the `which.max()` function, interval 835, on average, contains the maximum number of steps throughout the day.

&nbsp;


## Imputing missing values


```r
totalna <- sum(!complete.cases(activitydata))
```

There are a total of 2304 observations that contain missing values `(NA-s)` for the number of steps in the data set. While in the analysis so far we have just removed those observations from the data set, we can impute the missing data using a simple means approach. Basically, each missing value for an interval on any day is replaced with the average number of steps for that interval of the day, across all days. The averages were already calculated above and a `for loop` was used to replace the `NA-s`.


```r
completedata <- activitydata
for(i in 1:length(completedata$steps)) { 
    if (is.na(completedata$steps[i])) { 
        completedata$steps[i] <- 
            intervalstepssummary$steps[intervalstepssummary$interval == completedata$interval[i]] 
    } 
}
```

From the new data frame `completedata` we can use the same technique as before and plot a histogram of the total number of steps taken each day.


```r
dailystepsimputed <- summarise(group_by(completedata, date), dailysteps = sum(steps))
meanimputeddailysteps <- mean(dailystepsimputed$dailysteps)
medianimputeddailysteps <- median(dailystepsimputed$dailysteps)
imputedhist <- ggplot(dailystepsimputed, aes(x=dailysteps))
imputedhist + 
    geom_histogram(col="black", fill="orange", binwidth = 1000) + 
    labs(title="Histogram of total steps per day (imputed data)", x="Total steps (per day)", y="Number of days")
```

![](PA1_template_files/figure-html/imputedgraph-1.png) 

The mean for the total number of steps per day is 10766.19 and the median is
10766.19. The mean is the same as in the clean data set, but the median in the imputed data set is slightly larger than in the clean data. We can also put the two histograms on the same plot to compare the differences in the total daily number of steps.


```r
naremoved <- mutate(dailystepssummary, category = "na_removed")
imputed <- mutate(dailystepsimputed, category = "imputed")
comparison <- rbind(naremoved, imputed)
comparisonhist <- ggplot(comparison, aes(x=dailysteps, fill=category))
comparisonhist + 
    geom_histogram(binwidth = 1000, position = "dodge", col="black") +
    labs(title="Comparison of total daily steps - imputing vs. removing NA", x="Total steps (per day)", y="Number of days")
```

![](PA1_template_files/figure-html/imputationcomparison-1.png) 


&nbsp;

## Are there differences in activity patterns between weekdays and weekends?

We introduce a new factor describing the day of the week (weekday vs weekend).


```r
dayfactor <- factor(ifelse(weekdays(completedata$date) %in% c("Saturday", "Sunday"), "weekend", "weekday"))
completedata <- mutate(completedata, daytype = dayfactor)
weeksummary <- summarise(group_by(completedata, interval, daytype), steps = mean(steps))
```

We can then plot a time series of the average number of steps taken for each interval along the day, just like we did above, comparing the daily activity on weekdays agains weekends. There are some apparent differences in the peak distibutions and the total number of steps, as seen in the graphs.


```r
library(lattice)
xyplot(steps ~ interval | daytype, data = weeksummary, type="l", layout = c(1,2), main = "Average daily activity pattern - weekend vs. weekday", xlab = "Interval", ylab = "Number of steps")
```

![](PA1_template_files/figure-html/weekdaysplot-1.png) 

&nbsp;
