# Reproducible Research: Peer Assessment 1




## Loading and preprocessing the data

This is the R Code for loading the Data used for this Peer Assessment.
(please feel free and change the path)


```r
myD <- setwd("C:/Users/rosenranz/Documents/_coursera/src/RepData_PeerAssessment1")
filename <- "activity"
if(!file.exists(paste0(filename,".csv"))){
        unzip(paste0(filename,".zip"))
}        
o.data <- read.csv2(paste0(filename,".csv"), 
                    sep = ",", na.strings = "NA", 
                    header = TRUE, stringsAsFactors=FALSE)
```


Set the scientific notation off. Store the original scipen option value.


```r
scipen.value <- getOption("scipen")
options(scipen=20)
```


## What is mean total number of steps taken per day?

For this part of the assignment, we can ignore the missing values in the dataset.
Now, before we can plot a histogramm, we have to aggregate the original data to get the total number of steps taken each day.


```r
agg.data.all <- with(o.data, aggregate(steps, 
                                   list(Date = factor(date)), 
                                   sum))
```

To plot the histogram, i use the ggplot2 package. For the histogram plot i used the default binwidth.


```r
library(ggplot2)
p <- ggplot(agg.data.all, aes(x)) + geom_histogram(aes(fill = ..count..))
p <- p + labs(list(x="Number of steps per Day", y="", title="Total number of steps taken each day"))
print(p)
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk plot_hist_all](figure/plot_hist_all.png) 

Calculate the mean and median total number of steps taken per day


```r
(c.mean <- mean(agg.data.all$x, na.rm = TRUE))
```

```
## [1] 10766
```

```r
(c.median <- median(agg.data.all$x, na.rm = TRUE))
```

```
## [1] 10765
```

So, the mean of total number of steps taken per day is **10766.1887** and the median **10765**.


## What is the average daily activity pattern?

Before we are plottig a diagram of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis), we have to aggregate only the complete cases from the original data set.


```r
agg.data <- with(o.data[complete.cases(o.data), ], aggregate(steps, 
                                   list(Interval = factor(interval)), 
                                   mean))
```


Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
mmax <- agg.data[agg.data$x == max(agg.data$x), ]
# extract 5-minute interval on the aggregated data that contains the maximum number of steps
maxint <- as.numeric(as.character(mmax[1,1]))
# extract the number of steps
maxstep <- mmax[1,2]
```

After aggregate the data and calculating the maximum value, we can plot the average daily activity pattern.


```r
p <- ggplot(agg.data, aes(x=as.numeric(as.character(Interval)), y=x)) 
p <- p + geom_line(colour="blue")
p <- p + geom_vline(xintercept = maxint, alpha=.4)
p <- p + labs(list(x="5 min Intervall", y="average number of steps averaged across all days", 
   title="5-minute interval and the average number of steps taken,\n averaged across all days\n (October 2012 to November 2012)"))
print(p)
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

You can see the maximum value where the vertical line hits/intercepts the x-axis. So the **835** 5-minute intervall, on average across all the days in the dataset, contains the maximum number of steps. This interval contains **206.1698** average steps.


## Imputing missing values

In this dataset, the missing values are represented by the symbol NA (not available). We calculate the total number of missing values in the dataset (i.e. the total number of rows with NAs)



```r
(sum.na <- sum(is.na(o.data$steps)))
```

```
## [1] 2304
```

The total number of missing values in the steps variable of the original data is **2304**

Till now we have excluded the missing values. Here i want to apply an easy-to-use missing data imputation strategy. 

- First of all i will calculate the mean for every 5-minute interval across all days (df.means).

- Then i replace all the NA values in the variable "step" with the corresponding mean-value for that 5-minute intervall.

- at the end we have a new created dataset that is equal to the original dataset but with the missing data filled in.


```r
# aggregating the orignal data to calculate the mean for every 5-minutes interval
df.means <- with(o.data[complete.cases(o.data), ], aggregate(steps, 
                                                             list(interval = factor(interval)), 
                                                             mean))
# merging the data by the variable interval to generate a new dataset with the the new mean value corresponding to the interval.
data <- merge(o.data, df.means, by="interval", sort=FALSE)
# reorder the merged dataset by date and interval
data <- data[order(data$date, data$interval), ]
# inputing/replacing the new meanvalues only for the NA values in steps.
# the warning has no effect to our replacement !
data$steps[is.na(data$steps)] <- data$x
```

```
## Warning: number of items to replace is not a multiple of replacement
## length
```

```r
# reordering the rownames in the dataset
row.names(data) <- 1:nrow(data)
# delete the mean column from the dataset. not used anymore
data$x <- NULL
# reorder the column in the dataset.
data <- data[, c(2,3,1)]
```

Make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day.


```r
# aggregate data by Date and sum up the steps
agg.data.n <- with(data, aggregate(steps, 
                                 list(Date = factor(date)), 
                                 sum))
p <- ggplot(agg.data.n, aes(x)) + geom_histogram(aes(fill = ..count..))
p <- p + labs(list(x="Number of steps per Day", y="", 
                   title="Total number of steps taken each day.\n [ inpute missing values ]"))
print(p)
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

Calculate the mean and median total number of steps taken per day with the new created dataset without NAs.


```r
(c.mean.ip <- mean(agg.data.n$x))
```

```
## [1] 10766
```

```r
(c.median.ip <- median(agg.data.n$x))
```

```
## [1] 10766
```

The mean of total number of steps taken per day is **10766.1887** and the median **10766.1887** .  

  
Do these values differ from the estimates from the first part of the assignment?

Due to the fact that you can add a calculated mean of a vector to that same vector as many times you want, this doesnt change the mean value of this vector.
So there is no effect on the mean value.
And because of new values in the dataset (not excluded NAs and replaced it with the corresponding mean value) there is a slightly different median. Also due to the inputing process, the step values changed from integer to decimal. So the different between the median without NAs and the median with inputed NAs is -1.1887


```r
c.median-c.median.ip
```

```
## [1] -1.189
```

What is the impact of imputing missing data on the estimates of the total
daily number of steps?

1. more step values and therefore a higher value on the y-axes in the historgram (count/frequency)

2. a slightly different median (more values and all vaules in decimal, see above)


## Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
# new column ddate: date column to POSIXlt objects
data$ddate <- strptime(o.data$date, "%Y-%m-%d")
# generate a factor column with weedend and weekday
data$weekendday <- as.factor(ifelse(weekdays(data$ddate, abbreviate=TRUE) %in% c("Sa","So"), "weekend", "weekday"))
```

Now we can make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)


```r
# aggregate data for plotting. caluclating the mean for each intervall for weekdays and weekends
agg.data.wd <- with(data, aggregate(steps, 
                                 list(Interval = factor(interval), weekendday = weekendday), 
                                 mean))
# generating plot
p <- ggplot(agg.data.wd, aes(x=as.numeric(as.character(Interval)), y=x))
p <- p + geom_line()
p <- p + facet_grid(weekendday ~ .)
p <- p + labs(list(x="5 min Intervall", y="average number of steps averaged across all days", 
                   title="5-minute interval and the average number of steps taken,\n averaged across all days, seperated in Weekdays and Weekends\n (October 2012 to November 2012)"))
print(p)
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13.png) 

Switch back to the original scipen value (scientific notation on)


```r
options(scipen=scipen.value)
```
