---
title: "Reproducible Research: Peer Assessment 1"
author: "Ines L Breda"
date: "June 4, 2019"
output: 
 html_document:
    keep_md: true
---



## Loading and preprocessing the data

Link to data available in the course website. The **data** was loaded to R using *read.csv()* function from the file **activity.csv** located in the folder **data** placed in the working directory. Preprocessing data by formating the variable **date** to a date format variable.


```r
data <- read.csv(file = "./data/activity.csv")
data$date <- as.Date(data$date)
```


## What is mean total number of steps taken per day?

The function *aggregate()* was used to sum the number of steps by day. The output was stored in a table called **stepsdaily**. The NA-values present in **stepsdaily** were ignored.

The *ggplot* plot system (**ggplot2**) was used to make an histogram of the steps taken daily. 


```r
stepsdaily <- aggregate(data["steps"], by=data["date"], sum)
stepsdaily <- stepsdaily[complete.cases(stepsdaily),]

library(ggplot2)
ggplot(data = stepsdaily, aes(x = steps)) +
        geom_histogram(binwidth = 3000, color = "black", fill = "navyblue") +
        theme_bw()
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

The total count of daily steps mean is 1.0766189\times 10^{4} and the median is 10765.

## What is the average daily activity pattern?

The function *aggregate()* was used to average the number of steps by interval. The output was stored in a table called **stepsinterval**. The NA-values were again ignored.

The *ggplot* plot system (**ggplot2**) was used to make a time series of the average steps taken at each interval. 


```r
dataremovedNA <- data[complete.cases(data),]
stepsinterval <- aggregate (dataremovedNA["steps"], by = dataremovedNA["interval"], mean)

ggplot(stepsinterval, aes(x = interval, y = steps)) +
        geom_line(color = "navyblue") +
        theme_bw()
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

The interval with most steps in average accounting for all days is 835 

## Imputing missing values

The total number of rows with NA values in the **data** is 2304.

The NA values were all in the **steps** variable of the **data**. A new dataset was created (**datafilled**), in which the NA values of **data** were replaced by the mean number of steps at the given **interval**.


```r
datafilled <- transform(data, steps = ifelse(is.na(data$steps),
        stepsinterval$steps[match(data$interval,stepsinterval$interval)], 
        data$steps))
```

The *ggplot* plot system (**ggplot2**) was used to make an histogram of the steps taken daily based on the new dataset **datafilled**. 


```r
stepsdailyfilled <- aggregate(datafilled["steps"], by=datafilled["date"], sum)

ggplot(data = stepsdailyfilled, aes(x = steps)) +
        geom_histogram(binwidth = 3000, color = "black", fill = "navyblue") +
        theme_bw()
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

Replacing the NA values in the variable **steps** by the mean number of steps at the given **interval** changed the initial **data**. The datasets (with and without replacement of NA values) show no difference in the mean of steps (´r mean(stepsdailyfilled$steps) - mean(stepsdaily$steps, na.rm = TRUE)´). In opossite, the datasets show a difference in the median (´r median(stepsdailyfilled$steps) - median(stepsdaily$steps, na.rm = TRUE)´) and in the total count of steps (´r sum(stepsdailyfilled$steps) - sum(stepsdaily$steps)´).

## Are there differences in activity patterns between weekdays and weekends?


```r
datafilled$weekday <- ifelse(weekdays(datafilled$date) %in% c("Saturday", "Sunday"), "weekend", "weekday")

stepsintervalfilled <- aggregate (steps ~ interval + weekday, datafilled, mean)

ggplot(stepsintervalfilled, aes(x = interval, y = steps)) +
        geom_line(color = "navyblue") +
        theme_bw() +
        facet_grid(weekday~.) +
        theme_bw() +
        ylab("Average number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

