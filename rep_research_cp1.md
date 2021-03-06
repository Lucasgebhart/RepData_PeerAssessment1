This file conducts exploratory analysis of activity data.
---------------------------------------------------------

Begin by reading in the data

    data <- read.csv("activity.csv")
    head(data)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

The data is every five minutes. Let’s look at daily data

    dailysteps <- aggregate(steps~date, data, sum)

Histogram of the steps data
---------------------------

Sometimes it helps to see a quick histogram.

![](rep_research_cp1_files/figure-markdown_strict/data-1.png)

The data appears pretty close to normally distributed, with a number of
days where the person is inactive.

    mean(dailysteps$steps, na.rm= TRUE)

    ## [1] 10766.19

    median(dailysteps$steps, na.rm= TRUE)

    ## [1] 10765

Clearly, a lot of data is missing and it is possible many of these 0s
are incorrect. Let’s look at the data on an interval basis, instead of a
daily basis.

    meanintervalsteps <- aggregate(steps~interval, data, mean)

    plot(meanintervalsteps$interval, meanintervalsteps$steps, type = "l", main ="Average Step per Interval", xlab="5 minute interval", ylab="Mean Steps")

![](rep_research_cp1_files/figure-markdown_strict/unnamed-chunk-4-1.png)

There is a big spike near 800-900 minutes into the day, let’s find out
what interval this is.

    moststeps <- max(meanintervalsteps$steps)
    maxinterval <- data[meanintervalsteps$steps == moststeps,]
    maxinterval[1,3]

    ## [1] 835

The most steps occur around 835 minutes into a day. Maybe the person
likes afternoon runs? Now, let’s deal with missing data… there are a few
strategies of varying accuracy and ease of implementation. One option
would be to take the average of the interval before and after the
interval in question. I could easily implement this in excel… but have
insufficent R skills. The next option would be to give make the missing
data the average. But there are clearly different activity patterns. So
let’s fill the missing data with the average FOR THAT INTERVAL.

    missingdata <- is.na(data$steps)
    head(missingdata)

    ## [1] TRUE TRUE TRUE TRUE TRUE TRUE

    data1<-data[is.na(data$steps),]
    head(data1)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

    impute_int_avg <- meanintervalsteps$steps[match(data$interval, meanintervalsteps$interval)]
    steps_replace <- transform(data, steps = ifelse(is.na(data$steps), yes = impute_int_avg, no = data$steps))

    dailysteps_replace <- aggregate(steps~date, steps_replace, sum)

Let’s see what the data looks like after replacing all of the missing
values.

    hist(dailysteps_replace$steps, main = "Histogram of Daily Steps with imputed steps", xlab="Daily Steps", ylab="Frequency")

![](rep_research_cp1_files/figure-markdown_strict/unnamed-chunk-7-1.png)
Let’s see if there is a difference in activity between weekends and
weekdays. We’ll use data without imputed steps.

    #create the identifying column
    data$weekday <- weekdays(as.Date(data$date))
    data$weekend <- ifelse(data$weekday == "Saturday" | data$weekday == "Sunday", "Weekend", "Weekday")
    head(data)

    ##   steps       date interval weekday weekend
    ## 1    NA 2012-10-01        0  Monday Weekday
    ## 2    NA 2012-10-01        5  Monday Weekday
    ## 3    NA 2012-10-01       10  Monday Weekday
    ## 4    NA 2012-10-01       15  Monday Weekday
    ## 5    NA 2012-10-01       20  Monday Weekday
    ## 6    NA 2012-10-01       25  Monday Weekday

    typemeanintervalsteps <- aggregate(steps~interval + weekend, data, mean)
    View(typemeanintervalsteps)
    weekendsteps <- subset(typemeanintervalsteps, weekend == "Weekend")
    weekdaysteps <- subset(typemeanintervalsteps, weekend == "Weekday")



    library(ggplot2)
    plot2 <- ggplot()+
      geom_line(aes(x= typemeanintervalsteps$interval, y= typemeanintervalsteps$steps, group = typemeanintervalsteps$weekend, color = typemeanintervalsteps$weekend))+
      labs( x= "Interval",
            y= "Steps",
            color= "Legend")
    plot2

![](rep_research_cp1_files/figure-markdown_strict/unnamed-chunk-8-1.png)
