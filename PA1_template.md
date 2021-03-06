# Reproducible Research: Peer Assessment 1

Activity monitoring data assignment
===================================

First we are going to load the data previously downloaded from the github repo (https://github.com/rdpeng/RepData_PeerAssessment1) into the working directory. 



```r
data <- read.csv("activity.csv")
```



###What is mean total number of steps taken per day?

In order to calculate the total number of steps taken per day we need first to create a new dataframe where we sum the number of steps from the original data frame by date. 


```r
require(data.table)
```

```
## Loading required package: data.table
```

```r
dt<-data.table(data)[,sum(steps),by=date]
```


Here's a histogram of the total number of steps taken each day:

```r
hist(dt$V1,xlab='Sum of steps',ylab='Number of days',main='Histogram of total number of steps taken each day',col='blue')
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 


We calculate the mean and median of the total number of steps taken per day:

```r
mean(dt[,dt$V1],na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(dt[,dt$V1],na.rm=TRUE)
```

```
## [1] 10765
```



###What is the average daily activity pattern?

Here's a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis). As you can see below, we first remove the NA values.


```r
data2 <- na.omit(data)
dt2<-data.table(data2)[,mean(steps),by=interval]
plot(dt2,type="l",main='Average number of steps per 5-min interval across all days',ylab='number of steps',col='blue')
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 


I'm also calculating which interval is on average the one that contains the maximum number of steps: 835

```r
dt2[dt2$V1==max(dt2$V1)]
```

```
##    interval       V1
## 1:      835 206.1698
```



###Imputing missing values

We are going to calculate first how many missing values (coded as NA) we have in the data frame: 2304


```r
sum(is.na(data))
```

```
## [1] 2304
```


In order to impust these missing values, we decided to fill in all of those values with the mean of that 5-minute interval. To that end, we take the means already calculated in the dt2 data frame and insert them into a new dataset (data3). 


```r
data3 <- data

for (i in 1:nrow(data3)) 
        if (is.na(data3[i,"steps"])) {
                interval_day <- data3[i,"interval"]
                data3[i,"steps"]<-dt2[dt2$interval==interval_day,V1]
        }
```


Then we calculate the total number of steps taken each day by using the data from data3 and save it into dt3:


```r
dt3<-data.table(data3)[,sum(steps),by=date]
```


Next, we make a histogram of this new total number of steps taken each day from the data in dt3:


```r
hist(dt3$V1,xlab='Sum of steps',ylab='Number of days',main='Histogram of total number of steps taken each day',col='blue')
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 


By taking the mean and median, we see that the values differ very little from what we calculated above. The mean is still the same (just because we filled in the missing values using the mean per interval), whereas the median has only been altered from 10765 to 10766.19. Therefore we can consider that the impact of imputing missing data didn't impact negatively the estimates of the total daily number of steps. Actually, if you compare the 2 histograms you can see that the distribution is fairly identical, and the only difference is that more days are included into the 2nd histogram simply because there's more data points.


```r
mean(dt3[,dt3$V1])
```

```
## [1] 10766.19
```

```r
median(dt3[,dt3$V1])
```

```
## [1] 10766.19
```



###Are there differences in activity patterns between weekdays and weekends?

Taking the previous dataset with the filled-in missing values, we are going to replace the dates with the day of the week so we can distinguish between weekdays and weekends:


```r
data4 <- data3
data4[,"date"] <- weekdays(as.Date(data4[,"date"]))
data4[data4[,"date"] =="Saturday" | data4[,"date"] == "Sunday","date"] <- "weekend"
data4[data4[,"date"] != "weekend" ,"date"] <- "weekday"
```


Now we want to create a new dataframe with the mean of steps per day, differentiating between weekday or weekend. 


```r
dt4<-data.table(data4)[,mean(steps),by=c("interval","date")]
```


Finally we can make a panel plot containing a time series plot of the 5-minutes interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). For that we are going to use lattice in order to replicate the type of figure expected for this assignment.


```r
library(lattice)
xyplot(V1 ~ interval | date, data = dt4, type = "a", layout = c(1,2), ylab = 'number of steps')
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 


By comparing both plots we can determine that this individual is more active during the weekend than during the rest of the week, although there's some intervals in the morning where the average number of steps from Monday to Friday are slightly higher than those in the weekend.
