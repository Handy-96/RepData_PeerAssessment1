\#\# Loading and preprocessing the data

    library("data.table")
    library(ggplot2)
    fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    download.file(fileUrl, destfile = paste0(getwd(), '/repdata%2Fdata%2Factivity.zip'), method = "curl")
    unzip("repdata%2Fdata%2Factivity.zip",exdir = "data")

Read CSV
--------

    Data1 <- data.table::fread(input = "data/activity.csv")

What is mean total number of steps taken per day?
-------------------------------------------------

    Total_Steps <- Data1[, c(lapply(.SD, sum, na.rm = FALSE)), .SDcols = c("steps"), by = .(date)] 
    head(Total_Steps, 10)

    ##           date steps
    ##  1: 2012-10-01    NA
    ##  2: 2012-10-02   126
    ##  3: 2012-10-03 11352
    ##  4: 2012-10-04 12116
    ##  5: 2012-10-05 13294
    ##  6: 2012-10-06 15420
    ##  7: 2012-10-07 11015
    ##  8: 2012-10-08    NA
    ##  9: 2012-10-09 12811
    ## 10: 2012-10-10  9900

Histogram
---------

    ggplot(Total_Steps, aes(x = steps)) +
      geom_histogram(fill = "blue", binwidth = 1000) +
      labs(title = "Daily Steps", x = "Steps", y = "Frequency")

    ## Warning: Removed 8 rows containing non-finite values (stat_bin).

![](PA1_tempalte_files/figure-markdown_strict/unnamed-chunk-4-1.png)
\#\# Mean & Median

    Total_Steps[, .(Mean = mean(steps, na.rm = TRUE), Median = median(steps, na.rm = TRUE))]

    ##        Mean Median
    ## 1: 10766.19  10765

What is the average daily activity pattern?
-------------------------------------------

    Interval_5min <- Data1[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval)] 
    ggplot(Interval_5min, aes(x = interval , y = steps)) + geom_line(color="blue", size=1) + labs(title = "Avg. Daily Steps", x = "Interval", y = "Avg. Steps per day")

![](PA1_tempalte_files/figure-markdown_strict/unnamed-chunk-6-1.png)

5 minute itnerval containt max numbers
--------------------------------------

    Data1[steps == max(steps), .(max_interval = interval)]

    ## Empty data.table (0 rows and 1 cols): max_interval

Imputing missing values
-----------------------

    Data1[is.na(steps), .N ]

    ## [1] 2304

Filling missing values
----------------------

    Data1[is.na(steps), "steps"] <- Data1[, c(lapply(.SD, median, na.rm = TRUE)), .SDcols = c("steps")]

New Dataset
-----------

    data.table::fwrite(x = Data1, file = "data/tidyData.csv", quote = FALSE)

Histogram
---------

    Total_Steps <- Data1[, c(lapply(.SD, sum)), .SDcols = c("steps"), by = .(date)] 
    Total_Steps[, .(Mean_Steps = mean(steps), Median_Steps = median(steps))]

    ##    Mean_Steps Median_Steps
    ## 1:    9354.23        10395

    ggplot(Total_Steps, aes(x = steps)) + geom_histogram(fill = "blue", binwidth = 1000) + labs(title = "Daily Steps", x = "Steps", y = "Frequency")

![](PA1_tempalte_files/figure-markdown_strict/unnamed-chunk-11-1.png)

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

    Data1 <- data.table::fread(input = "data/activity.csv")
    Data1[, date := as.POSIXct(date, format = "%Y-%m-%d")]
    Data1[, `Day of Week`:= weekdays(x = date)]
    Data1[grepl(pattern = "Monday|Tuesday|Wednesday|Thursday|Friday", x = `Day of Week`), "weekday or weekend"] <- "weekday"
    Data1[grepl(pattern = "Saturday|Sunday", x = `Day of Week`), "weekday or weekend"] <- "weekend"
    Data1[, `weekday or weekend` := as.factor(`weekday or weekend`)]
    head(Data1, 10)

    ##     steps       date interval Day of Week weekday or weekend
    ##  1:    NA 2012-10-01        0      Monday            weekday
    ##  2:    NA 2012-10-01        5      Monday            weekday
    ##  3:    NA 2012-10-01       10      Monday            weekday
    ##  4:    NA 2012-10-01       15      Monday            weekday
    ##  5:    NA 2012-10-01       20      Monday            weekday
    ##  6:    NA 2012-10-01       25      Monday            weekday
    ##  7:    NA 2012-10-01       30      Monday            weekday
    ##  8:    NA 2012-10-01       35      Monday            weekday
    ##  9:    NA 2012-10-01       40      Monday            weekday
    ## 10:    NA 2012-10-01       45      Monday            weekday

Panel plot
----------

    Data1[is.na(steps), "steps"] <- Data1[, c(lapply(.SD, median, na.rm = TRUE)), .SDcols = c("steps")]
    IntervalDT <- Data1[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval, `weekday or weekend`)] 
    ggplot(IntervalDT , aes(x = interval , y = steps, color=`weekday or weekend`)) + geom_line() + labs(title = "Avg. Daily Steps by Weektype", x = "Interval", y = "No. of Steps") + facet_wrap(~`weekday or weekend` , ncol = 1, nrow=2)

![](PA1_tempalte_files/figure-markdown_strict/unnamed-chunk-13-1.png)
