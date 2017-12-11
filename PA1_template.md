### Downloading and opening data

This document will download and analyze the [Activity Monitoring
Data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip).
First, set working directory and check for existence of the .zip file.
Second, unzip the file to get access to activity.csv. Third, open csv
and look at the data set.

    wd <- "/Users/romanbraun/Dropbox (Privat)/Coursera/Data Science/5_reproducible_research/W2 - assignment"
    setwd(wd)
    if (file.exists("repdata_data_activity.zip")) {
            unzip("repdata_data_activity.zip")
    } else {
            download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", 
            "repdata_data_activity.zip")
            unzip("repdata_data_activity.zip")
    }
    df <- read.table("activity.csv", sep=",", header = TRUE)  
    #head(df)
    #dim(df)
    #as.Date(df$date[2])
    df$date <- as.Date(df$date)

    ## Warning in strptime(xx, f <- "%Y-%m-%d", tz = "GMT"): unknown timezone
    ## 'zone/tz/2017c.1.0/zoneinfo/America/New_York'

### Calculating mean number of steps per day

You can also embed plots, for example:

    steps_sum <- aggregate(steps ~ date, data = df, sum) 
    #dim(steps_sum)
    #steps_sum
    # Print steps per day as list
    #steps_sum
    library(ggplot2)
    qplot(x = steps_sum$steps, xlab = "steps", binwidth = 500)

![](PA1_template_files/figure-markdown_strict/steps%20per%20day%20plot-1.png)

Figure 1: Distribution of sum of steps in dataset

Mean steps per day:

    mean(steps_sum$steps)

    ## [1] 10766.19

Median steps per day:

    median(steps_sum$steps)

    ## [1] 10765

### What is the daily pattern?

    #head(df)
    #class(df)
    steps_int <- aggregate(df, by=list(df$interval), FUN=mean, na.rm=TRUE) 
    #steps_int
    ggplot(steps_int, aes(x = interval, y = steps)) +
            geom_line(stat="identity")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

Figure 2: Daily pattern of steps throughout data set

Maximum of steps

    steps_int[which.max(steps_int$steps),c(1,2)]

    ##     Group.1    steps
    ## 104     835 206.1698

### Imputing missing values

### Removing NA values

Replace NA values by '0'.

    dfnew <- df
    dfnew$steps[is.na(dfnew$steps)] <- 0
    sum(is.na(dfnew$steps))

    ## [1] 0

Histogram of total number of steps including imputed values:

    steps_sum <- aggregate(steps ~ date, data = dfnew, sum)
    library(ggplot2)
    qplot(x = steps_sum$steps, xlab = "steps", binwidth = 500)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-6-1.png)

Figure 3: Distribution of number of steps after imputing missing values

Mean steps per day with imputed values:

    mean(steps_sum$steps)

    ## [1] 9354.23

Median steps per day with imputed values:

    median(steps_sum$steps)

    ## [1] 10395

### Are there differences in activity patterns between weekdays and weekends?

    df$weekday <- weekdays(as.Date(df$date))
    mofr <- c("Montag", "Dienstag", "Mittwoch", "Donnerstag", "Freitag")
    saso <- c("Samstag", "Sonntag")
    df_week <- df[df$weekday == mofr,]

    ## Warning in df$weekday == mofr: Länge des längeren Objektes
    ##       ist kein Vielfaches der Länge des kürzeren Objektes

    df_week <- subset(df, weekday %in% mofr)
    df_wend <- subset(df, weekday %in% saso)
    sel <- c("steps", "interval")
    summary(df_week)

    ##      steps             date               interval        weekday         
    ##  Min.   :  0.00   Min.   :2012-10-01   Min.   :   0.0   Length:12960      
    ##  1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8   Class :character  
    ##  Median :  0.00   Median :2012-10-31   Median :1177.5   Mode  :character  
    ##  Mean   : 35.34   Mean   :2012-10-31   Mean   :1177.5                     
    ##  3rd Qu.:  8.00   3rd Qu.:2012-11-15   3rd Qu.:1766.2                     
    ##  Max.   :806.00   Max.   :2012-11-30   Max.   :2355.0                     
    ##  NA's   :1728

    df_week <- df_week[,sel]
    #head(steps_week)
    steps_week <- aggregate(df_week, by=list(df_week$interval), FUN=mean, na.rm=TRUE) 
    ggplot(steps_week, aes(x = interval, y = steps)) +
            geom_line(stat="identity")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-9-1.png)

Figure 4: Steps pattern on week days (Monday - Friday).

    df_wend <- df_wend[,sel]
    steps_wend <- aggregate(df_wend, by=list(df_wend$interval), FUN=mean, na.rm=TRUE) 
    #steps_int
    ggplot(steps_wend, aes(x = interval, y = steps)) +
            geom_line(stat="identity")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-10-1.png)

Figure 5: Steps pattern on weekend (Saturday / Sunday).

### Create new dataset

    write.table(dfnew, "activity_new.csv", sep =";", row.names = FALSE)
    steps_sum_new <- aggregate(steps ~ date, data = dfnew, sum) 
    dim(steps_sum_new)

    ## [1] 61  2

    # Print steps per day as list
    #steps_sum


    library(ggplot2)

    ggplot(steps_sum_new, aes(x=date, y=steps)) +
            geom_bar(stat = "identity")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-11-1.png)

### Conclusion

There's no difference in the histogram by imputing missing values,
because NA was replace by '0' which makes sense in this context.
However, the imputing of "0" had an effect on the calculation of the
mean and median number of steps per day. No data available means that
there were no steps performed. Analysis of steps pattern on weekdays and
weekends shows substantial differences.
