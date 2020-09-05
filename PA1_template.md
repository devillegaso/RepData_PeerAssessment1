Loading and preprocessing the data
----------------------------------

To begin with the analysis with necessary to read the data and transform
the column date to a “date type” so it could be better interpreted and
easy to work with.

    data=read.csv("activity.csv")
    summary(data)

    ##      steps            date              interval     
    ##  Min.   :  0.00   Length:17568       Min.   :   0.0  
    ##  1st Qu.:  0.00   Class :character   1st Qu.: 588.8  
    ##  Median :  0.00   Mode  :character   Median :1177.5  
    ##  Mean   : 37.38                      Mean   :1177.5  
    ##  3rd Qu.: 12.00                      3rd Qu.:1766.2  
    ##  Max.   :806.00                      Max.   :2355.0  
    ##  NA's   :2304

    str(data)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

    data$date<-as.Date(as.character(data$date,'%Y%m%d'))

What is mean total number of steps taken per day?
-------------------------------------------------

For this part, NA values were ignored. First, the total number of steps
were calulated for each day of the analysis and a histogram was plotted.
After that the mean and median of the daily number of steps were
calculated too.

    dailytotal<-aggregate(data$steps,list(date=data$date),sum, na.rm=TRUE)
    hist(dailytotal$x,breaks=nrow(dailytotal),xlab="Total daily number of steps",main="Histogram of total daily number of steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)

    dailymean<-aggregate(data$steps,list(date=data$date),mean, na.rm=TRUE)
    dailymedian<-aggregate(data$steps,list(date=data$date),median, na.rm=TRUE)

Now, both mean and median will be plotted so they can be more easily
interpreted

    par(mfcol=c(2,1))
    hist(dailymean$x,breaks=nrow(dailymean),xlab="Daily mean number of steps",main="Histogram of daily mean number of steps")
    hist(dailymedian$x,breaks=nrow(dailymedian),xlab="Daily median number of steps",main="Histogram of daily median number of steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

What is the average daily activity pattern?
-------------------------------------------

To calculate the five minute interval mean across the days the aggregate
function was used as it follows. Besides, the interval which corresponds
to the maximum number of steps was determined too.

    fiveminutemean<-aggregate(data$steps,list(interval=data$interval),mean,na.rm=TRUE)
    maxinterval<-fiveminutemean$interval[which(fiveminutemean$x==max(fiveminutemean$x))]
    print(c("The interval with the maximum mean of steps is: ",maxinterval))

    ## [1] "The interval with the maximum mean of steps is: "
    ## [2] "835"

Now for plotting:

    with(fiveminutemean,plot(interval,x,type="l", ylab= "Average number of steps across all days",xlab="5 minute interval",col='deepskyblue4'))
    abline(v=maxinterval,col="red")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-5-1.png)

Imputing missing values
-----------------------

Calculating the total number of missing values in the dataset:

    totalmissing<-sum(is.na(data$steps))
    totalmissing

    ## [1] 2304

To fill all missing values, they were each replaced by 0, this decission
was made due to the fact that 0 was a common value for many of the
observations in the original dataset

    data2<-data
    data2$steps[is.na(data2$steps)]<-0

Now histograms are created to show the new mean and median for the
dataset, first the mean is shown, then the median, they are both
compared with the original values

    dailymean2<-aggregate(data2$steps,list(date=data$date),mean, na.rm=TRUE)
    dailytotal2<-aggregate(data2$steps,list(date=data$date),sum, na.rm=TRUE)
    dailymedian2<-aggregate(data2$steps,list(date=data$date),median, na.rm=TRUE)

    par(mfcol=c(2,1))
    hist(dailymean2$x,breaks=nrow(dailymean2),xlab="Daily mean number of steps",main="Histogram of daily mean number of steps NA replaced by 0")
    hist(dailymean$x,breaks=nrow(dailymean),xlab="Daily mean number of steps",main="Histogram of daily mean number of steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-8-1.png)

    par(mfcol=c(2,1))
    hist(dailymedian2$x,breaks=nrow(dailymedian2),xlab="Daily median number of steps",main="Histogram of daily median number of steps NA replaced by 0")
    hist(dailymedian$x,breaks=nrow(dailymedian),xlab="Daily median number of steps",main="Histogram of daily median number of steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-9-1.png)

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

First, a weekday column is created in the dataset using the weekdays()
function to calculate the corresponding day to each of the observation.
After that, a for loop transforms the days of the week to weeekday or
weekend depending on its value. Note that days are in Spanish, this is
due my regional configuration.

    data2$weekday<-weekdays(data2$date)
    n<-0
    for (i in data2$weekday){
      if (i=="lunes"| i=="martes"|i=="miércoles"|i=="jueves"|i=="viernes"){
        n<-n+1
        data2$weekday[n]<-"weekday"
      }else{
        n<-n+1
        data2$weekday[n]<-"weekend"
      }
    }

    data2$weekday<-as.factor(data2$weekday)

Now the average number steps per interval is calculated both for the
weekdays and weekends

    weekdays<-data2[data2$weekday=="weekday",]
    weekends<-data2[data2$weekday=="weekend",]
    averageweekday<-aggregate(weekdays$steps,list(interval=weekdays$interval),mean)
    averageweekend<-aggregate(weekends$steps,list(interval=weekends$interval),mean)

Finally, time series showing the variation of the average number of
steps for both weekdays and weekends are plotted.

    par(mfcol=c(2,1),mar=c(4,4,2,2))
    with(averageweekday,plot(interval,x,type="l",col='deepskyblue4',ylim=c(0,200),main="Average of steps for weekdays",xlab="Interval",ylab="Average number of steps"))
    with(averageweekend,plot(interval,x,type="l",col='deepskyblue4',ylim=c(0,200),main="Average of steps for weekends",xlab="Interval",ylab="Average number of steps"))

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-12-1.png)
