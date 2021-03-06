---
title: "Reproducible Research Week Two Assignment"
output: 
  html_document: 
    keep_md: yes
---



## 1 LOADING AND PREPROCESSING DATA
First I set the working directory to be the one in which the files are contained.  Then I read the file into R and download programs that will be helpful.


```r
setwd("~/Desktop/r")
orgCSV<-read.csv("activity.csv")
orgCSV2=orgCSV
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(ggplot2)
library(tidyr)
```

## FIND MEAN # of STEPS / DAY
Next, I take the original CSV file and group it by date so that I can find the average # of steps per day.  I then find this average with the "summarize" function and call it "total"


```r
CSV_by_date<-group_by(orgCSV,date)  
tsteps<-summarize(CSV_by_date,total_steps=sum(steps))
tsteps<-as.data.frame(tsteps)
```

Now make a histogram of this data, and find the mean and median steps/day.


```r
hist(tsteps[,2],xlab="Total Steps Taken",main="Steps Taken Per Day")
```

![](PA1_template_files/figure-html/Histogram, Mean, Median-1.png)<!-- -->

```r
meansteps<-mean(tsteps[,2],na.rm=TRUE)
medsteps<-median(tsteps[,2],na.rm=TRUE)
meansteps
```

```
## [1] 10766.19
```

```r
medsteps
```

```
## [1] 10765
```

## FIND AVERAGE DAILY ACTIVITY PATTERN
First, group by the interval variable.  Then summarize this by finding the mean of the steps taken during each time interval.  Plot this. Show time interval with greatest # of steps on average.


```r
CSV_by_time<-group_by(orgCSV,interval)
steps_by_time<-summarize(CSV_by_time,meansteps=mean(steps,na.rm=TRUE))
plot(steps_by_time$interval,steps_by_time$meansteps,type="l",xlab="Interval",ylab="Steps",main="Total Steps by Time Interval")
```

![](PA1_template_files/figure-html/Daily Activity Average-1.png)<!-- -->

```r
findmax<-which.max(steps_by_time$meansteps)
maxtime<-steps_by_time[findmax,1]
as.character(maxtime)
```

```
## [1] "835"
```

## 3 Imputing Missing Values
Find total number of NA value (which only appear in "steps" column).  Then take these NA values and replace them with the average of the corresponding time interval (e.g. if there is an NA value for time interval 120, find the average steps taken in all 120 time interval and plus in this number for the NA value).


```r
number_of_NA<-17568-sum(complete.cases(orgCSV))
NARow<-which(is.na(orgCSV$steps))
mrg<-merge(orgCSV2,steps_by_time,by="interval")
mrg<-merge(orgCSV2,steps_by_time,by="interval")
NARow2<-which(is.na(mrg[,2]))

for(i in 1:2403){
    replace_na(mrg[NARow2[i],2],mrg[NARow[i],4])
}

mrg_by_date<-group_by(mrg,date)  
tsteps2<-summarize(mrg_by_date,total_steps=sum(steps))
tsteps2<-as.data.frame(tsteps2)
```

Make a histogram of this data and then report the mean and median.  You will see that both the mean and the median lower slightly when NA values are imputed.


```r
hist(tsteps2[,2],xlab="Total Steps Taken",main="Steps Taken Per Day")
```

![](PA1_template_files/figure-html/Impute Part 2-1.png)<!-- -->

```r
meansteps2<-mean(tsteps2[,2],na.rm=TRUE)
medsteps2<-median(tsteps2[,2],na.rm=TRUE)
meansteps2
```

```
## [1] 10766.19
```

```r
medsteps2
```

```
## [1] 10765
```

### Are there differences in Activity Patterns?
We will see if there are differences in activity patterns between weekends and weekdays.  To do this, first we will edit mrg (the database with imputed values, i.e. the complete the database) so that we make a factor variable with results of "weekday" and "weekend".


```r
date<-as.character(mrg[,3])
date<-as.POSIXct(date,format="%Y-%m-%d")
mrg[,3]=date
mrg<-cbind(mrg,weekdays(mrg[,3]))
Day_Type<-factor(mrg[,5],levels=c("Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"),labels=c("weekend","weekday","weekday","weekday","weekday","weekday","weekend"))
mrg2<-cbind(mrg,Day_Type)
mrg2<-mrg2[,c(1:3,6),]
```

Group dataframe so as to be able to compare weekend days to weekdays. Graph this.


```r
mrg2Grouped<-group_by(mrg2,Day_Type,interval)
summary<-summarize(mrg2Grouped,mean=mean(steps))
ggplot(mrg2Grouped,aes(mrg2Grouped$interval,mrg2Grouped$steps))+facet_wrap(Day_Type)+geom_line()+labs(title="Day-Type Steps Comparison",x="Time Interval",y="Number of Steps")
```

```
## Warning: Removed 2 rows containing missing values (geom_path).
```

![](PA1_template_files/figure-html/Graph Day Type Comparison-1.png)<!-- -->











