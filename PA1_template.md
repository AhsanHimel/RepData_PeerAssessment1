---
title: "Reproducible Research: Peer Assessment 1"
author: Md Ahsanul Islam
output: 
  html_document:
    keep_md: true
    toc_float: true
    toc: true
---

---

**Setting some global options**


```r
knitr::opts_chunk$set(
  comment = "", 
  message=F, 
  warning = F
)
```

**Loading required packages**   


```r
packages <- c("ggplot2", "dplyr")

if("pacman" %in% rownames(installed.packages()) == F){
  install.packages("pacman")
  pacman::p_load(packages)
} else {
  pacman::p_load(packages, character.only = T)
}
```


## Loading and preprocessing the data

**Question:**  
Show any code that is needed to -

1. Load the data   
2. Process/transform the data (if necessary) into a format suitable for your analysis   

**Solution:**  

Loading the data:

```r
init.activity <- read.csv(unz("activity.zip", "activity.csv"), 
                     colClasses=c("numeric", "Date", "numeric"))
```


```r
summary(init.activity)
```

```
     steps             date               interval     
 Min.   :  0.00   Min.   :2012-10-01   Min.   :   0.0  
 1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8  
 Median :  0.00   Median :2012-10-31   Median :1177.5  
 Mean   : 37.38   Mean   :2012-10-31   Mean   :1177.5  
 3rd Qu.: 12.00   3rd Qu.:2012-11-15   3rd Qu.:1766.2  
 Max.   :806.00   Max.   :2012-11-30   Max.   :2355.0  
 NA's   :2304                                          
```


```r
activity <- subset(init.activity, !is.na(init.activity$steps))
summary(activity)
```

```
     steps             date               interval     
 Min.   :  0.00   Min.   :2012-10-02   Min.   :   0.0  
 1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8  
 Median :  0.00   Median :2012-10-29   Median :1177.5  
 Mean   : 37.38   Mean   :2012-10-30   Mean   :1177.5  
 3rd Qu.: 12.00   3rd Qu.:2012-11-16   3rd Qu.:1766.2  
 Max.   :806.00   Max.   :2012-11-29   Max.   :2355.0  
```



## What is mean total number of steps taken per day? {.tabset .tabset-fade .tabset-pills}

**Questions:**  
For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day
2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
3. Calculate and report the mean and median of the total number of steps taken per day

**Solutions:**   

### Answer 1

Calculating total number of steps taken per day - 

```r
perday.steps <- aggregate(steps ~ date, data = activity, FUN = sum)
head(perday.steps)
```

```
        date steps
1 2012-10-02   126
2 2012-10-03 11352
3 2012-10-04 12116
4 2012-10-05 13294
5 2012-10-06 15420
6 2012-10-07 11015
```

```r
summary(perday.steps)
```

```
      date                steps      
 Min.   :2012-10-02   Min.   :   41  
 1st Qu.:2012-10-16   1st Qu.: 8841  
 Median :2012-10-29   Median :10765  
 Mean   :2012-10-30   Mean   :10766  
 3rd Qu.:2012-11-16   3rd Qu.:13294  
 Max.   :2012-11-29   Max.   :21194  
```

### Answer 2

Making a histogram of the total number of steps taken per day -

```r
perday.steps %>% ggplot() +
  geom_histogram(aes(x = steps), boundary = 0, 
                 binwidth = 3000, col = "white", fill = "royalblue4") +
  scale_x_continuous(breaks = seq(0, 24000, 3000)) +
  scale_y_continuous(breaks = seq(0, 20, 3)) +
  theme_bw() +
  labs(title = "Histogram of steps taken per day",
       x = "Steps per day", y = "No. of days") +
  theme(plot.title = element_text(face = "bold", size = 15, hjust = 0.5))
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

### Answer 3

Calculating mean - 

```r
mean(perday.steps$steps)
```

```
[1] 10766.19
```
Calculating median - 

```r
median(perday.steps$steps)
```

```
[1] 10765
```

## What is the average daily activity pattern? {.tabset .tabset-fade .tabset-pills}

**Questions:**   

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

**Solutions:**   

### Answer 1


```r
activity %>% 
  group_by(interval) %>% 
  summarise(average = mean(steps)) %>% 
  ggplot() + 
  geom_line(aes(x = interval, y = average), col = "royalblue4") +
  scale_x_continuous(breaks = seq(0, 2400, 200)) +
  labs(title = "Average Steps Taken", subtitle = "Per time interval",
       x = "Time Interval", y = "Average steps") +
  theme(plot.title = element_text(face = "bold", size = 15, hjust = 0))
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

### Answer 2


```r
activity %>% 
  group_by(interval) %>% 
  summarise(average = mean(steps)) %>% 
  filter(average == max(average))
```

```
# A tibble: 1 x 2
  interval average
     <dbl>   <dbl>
1      835    206.
```

## Imputing missing values

**Questions:**   
Note that there are a number of days/intervals where there are missing values (coded as \color{red}{\verb|NA|}NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with \color{red}{\verb|NA|}NAs)
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

**Solutions:**   


## Are there differences in activity patterns between weekdays and weekends?
