Reproducible Research: Peer Assessment 1
=====



*Bishoy Sharobim*  
*2017-09-04*


<br>

### Part 1: Loading and prepocessing the data 

     
**(1) Load the data** 




```r
    a <- getwd()

    setwd(a)

    fileurl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"

    if(!file.exists("Dataset.zip")){
    download.file(fileurl, "./Dataset.zip")}
    
    if(!file.exists("activity.csv")){
    unzip(zipfile="Dataset.zip", exdir=".")}
    
    data <- read.csv("activity.csv")
    
    library(ggplot2)
```

<br>
    
    

**(2) Process/transform the data**



```r
    data1 <- data
    
    data1$'date' <- as.Date(data1$'date', format="%Y-%m-%d")
```

<br>


### Part 2: What is mean total number of steps taken per day?
**(1) Histogram of the total number of steps taken each day.**  



```r
    total <- aggregate(steps ~ date, data1, sum)
    
    hist(total$'steps', 
        breaks = 20, 
        col = "red",
        xlab = "Mean total number of steps taken per day",
        main = "Histogram of the total number of steps taken each day")
```

![](My_report_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

<br>



**(2) Calculate and report the mean and median total number of steps taken per day**


```r
    meansteps <- mean(total$'steps')
    mediansteps <- median(total$'steps')
```

The mean and median total number of steps taken per day is 1.0766189\times 10^{4} and 10765, respectively.

<br>

### Part 3: What is the average daily activity pattern?

**(1) Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).'**


```r
    meanperinterval <- aggregate(steps ~ interval , data1, mean)

    ggplot(meanperinterval, aes(interval, steps)) +
        geom_line() +
        xlab("5-minute interval") +
        ylab("Average number of steps taken") +
        ggtitle("Average number of steps taken per 5-min interval across all days") +
        theme(plot.title = element_text(hjust = 0.5))
```

![](My_report_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

<br>

**(2) Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?**

```r
    meanperinterval[which.max(meanperinterval$'steps'), ]
```

```
##     interval    steps
## 104      835 206.1698
```
  
The 835th interval which has a max number of steps of 206.1698113.

<br>

### Part 4: Input missing values

**(1) Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)**  


```r
    sum(is.na(data1))
```

```
## [1] 2304
```

Total number of missing values in the dataset is 2304. 

<br>

**(2) Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.**  

First I will try to replace the NA values with the mean number of total steps for the day for which a given NA value appears.


```r
    NAdataset <- data[is.na(data1), ]
    NAdataset$'date' <- as.Date(NAdataset$'date')    
        
    meanperday <- aggregate(steps ~ date, data1, mean)
    
    sum(meanperday$'date' == NAdataset$'date')
```

```
## Warning in `==.default`(meanperday$date, NAdataset$date): longer object
## length is not a multiple of shorter object length
```

```
## [1] 0
```

```r
    sum(NAdataset$'date' %in% meanperday$'date')
```

```
## [1] 0
```

This method cannot work because all missing values occur in all observations for a given day. A particular day has either recorded values for every single interval, or NA values for every single interval. 

Thus I am going to try to use the mean for every 5-min interval across all the days.


```r
    meanperinterval <- aggregate(steps ~ interval , data1, mean)
```

<br>

**(3) Create a new dataset that is equal to the original dataset but with the missing data filled in.**


```r
    NAdataset$'steps' <- meanperinterval$'steps'[meanperinterval$'interval' %in% NAdataset$'interval']
    
    data3 <- data1
    
    NAindices <- which(is.na(data1) == TRUE)
    
    data3$'steps'[NAindices] <- NAdataset$'steps'  
```

<br>

**(4)**  
***a) Make a histogram of the total number of steps taken each day.**  


```r
    total1 <- aggregate(steps ~ date, data3, sum)
    
    hist(total1$'steps', 
        breaks = 20, 
        col = "red",
        xlab = "Mean number of steps",
        main = "Histogram of the total number of steps taken each day")
```

![](My_report_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

<br>

**b) Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment?**  


```r
    meansteps1 <- mean(total1$'steps')
    mediansteps1 <- median(total1$'steps')
    
    data.frame("mean steps" = c(meansteps, meansteps1), "median steps" = c(mediansteps, mediansteps1))
```

```
##   mean.steps median.steps
## 1   10766.19     10765.00
## 2   10766.19     10766.19
```

The mean and median total number of steps taken per day is 1.0766189\times 10^{4} and 1.0766189\times 10^{4}, respectively. The mean values are the same, whilst the median steps differ extremely little.

<br>

**What is the impact of imputing missing data on the estimates of the total daily number of steps?**

This seems to highly depend on how you impute the missing data. Since I used the average for a given interval, there was practically no difference because we basically pulled the averages closer to the inserted average value.

<br>

### Part 5: Are there differences in activity patterns between weekdays and weekends? Use the dataset with the filled-in missing values for this part.
  
**Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.**  


```r
    weekday <- weekdays(data3$'date')

    weekdayfactor <- as.factor(weekday)
    
    levels(weekdayfactor) <- c("Weekday", "Weekday", "Weekend", "Weekend", "Weekday", "Weekday", "Weekday")
    
    data4 <- data3

    data4[, 4] <- levels(weekdayfactor)
    
    names(data4)[4] <- "Week_type"
```

<br>

**Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).**  

```r
    data5 <- aggregate(steps ~ interval + Week_type, data4, mean)
    
    ggplot(data5, aes(interval, steps)) +
        geom_line() +
        xlab("5-minute interval") +
        facet_grid(Week_type ~ . ) +
        ylab("Average number of steps taken") +
        ggtitle("Average number of steps by 5-min intervals (weekdays VS weekends)") +
        theme(plot.title = element_text(hjust = 0.5))
```

![](My_report_files/figure-html/unnamed-chunk-15-1.png)<!-- -->
    
