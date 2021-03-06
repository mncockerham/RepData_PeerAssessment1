Peer Assesment 1 Knitr and Analysis
========================================================

## Loading and preprocessing the data

To prepare for the data analys it is neccessary to load the raw file into R

The following code
 
 - assigns home directory
 - unzips the zipped file 
 - loads the raw file
 - creates a summary data set where the results are grouped by date and we store the total number of steps
 


```r
setwd <- "/Users/mcockerham/RepData_PeerAssessment1"
unzip("/Users/mcockerham/RepData_PeerAssessment1/activity.zip", exdir = "/Users/mcockerham/RepData_PeerAssessment1")
RawData <- read.csv("/Users/mcockerham/RepData_PeerAssessment1/activity.csv", 
    na.strings = "NA", header = TRUE)
summary <- aggregate(RawData$steps, by = list(RawData$date), FUN = sum, na.rm = TRUE)
```



## What is mean total number of steps taken per day?

We can then quicly generate a histogram showing the number of steps a day, this chart bins the step counts and gives us a distribution of the result set.



```r
hist(summary$x, col = "blue", xlab = "Steps", main = "Histogram of Steps per Day")
```

![plot of chunk Chart1](figure/Chart1.png) 


The values in the historgram have a mean and median value of:


```r
mean(summary$x)
```

```
## [1] 9354
```

```r
median(summary$x)
```

```
## [1] 10395
```



## What is the average daily activity pattern?

To see the average activy we need to create a dataset that shows activy by the 5 minute intervals.  These five minute intervals are stored in the interval column of the data set



```r
bytime <- aggregate(RawData$steps, by = list(RawData$interval), FUN = mean, 
    na.rm = TRUE)
```


As soon as the data set is created we can then graph the results


```r
plot(bytime, type = "l", ylab = "Steps", xlab = "Time in 5 Minute Increments", 
    main = "Steps by Time")
```

![plot of chunk Chart2](figure/Chart2.png) 



It is also possible to identify the row(s) with the maximume number of steps, in our data set the maximume number of steps was taken at the interval:


```r
bytime[bytime$x == max(bytime$x), ]$Group.1
```

```
## [1] 835
```



## Imputing missing values

Within the dataset we have a number of null values, one way to get a count is to use complete.cases to identify the nulls, by then using ! we can get a vector with 1 for each null value, this then can be simply summed up to get a count of null values.


```r
sum(!complete.cases(RawData$steps))
```

```
## [1] 2304
```


Since the order of intervals is the same in all cases, we can simply loop the vector of average valeus on the complete dataset this will give us the default value for any time period. (Average steps by increment we calculated earlier)


```r
RawData$rep <- bytime$x
```


While many ways exist to create this cleaned value, I perfer to create new columns and modify to allow for easier audits/comparisons between the different data sets.

- I created a new column called new that has the steps taken
- Then I used complete.cases to identify all the null values and replace them with the averarage number of steps taken(rep)
- lastly we can create a second data set that is a summary of stpes by date based on the (new) column instead of the (steps) column


```r
RawData$new <- RawData$steps
RawData[!complete.cases(RawData$steps), ]$new <- RawData[!complete.cases(RawData$steps), 
    ]$rep
summary2 <- aggregate(RawData$new, by = list(RawData$date), FUN = sum, na.rm = TRUE)
```


A new historgram can then be created with these values.


```r
hist(summary2$x, col = "red", xlab = "Steps", main = "Histogram of Steps per Day")
```

![plot of chunk Chart3](figure/Chart3.png) 


The values in the historgram have a mean and median value of:


```r
mean(summary2$x)
```

```
## [1] 10766
```

```r
median(summary2$x)
```

```
## [1] 10766
```




## Are there differences in activity patterns between weekdays and weekends?

To look at the difference between weekend and weekdays it is first necessary to identify what values represent a weekend and a weekday.
 
- I am creating a column that stores day of week using the weekdays function, to do this I need to also convert date to a date format
- next I am adding a column called Ind that will show if it is a weekday or weekend 
- 1 will be for weekend and I need to update all entries for both Saturday and Sunday  

I know have a data set that has both weekend and weekdays identified


```r
RawData$day <- weekdays(strptime(RawData$date, "%Y-%m-%d"))
RawData$Ind <- "Weekday"
RawData[RawData$day == "Sunday", ]$Ind <- "Weekend"
RawData[RawData$day == "Saturday", ]$Ind <- "Weekend"
byWeekday <- aggregate(RawData[RawData$Ind == "Weekday", ]$new, by = list(RawData[RawData$Ind == 
    "Weekday", ]$interval), FUN = mean, na.rm = TRUE)
byWeekEnd <- aggregate(RawData[RawData$Ind == "Weekend", ]$new, by = list(RawData[RawData$Ind == 
    "Weekend", ]$interval), FUN = mean, na.rm = TRUE)
```

 

```r
par(mfrow = c(2, 1))
par(cex = 0.6)
par(mar = c(1, 1, 1.5, 1))
par(col = "blue")
par(bg = "gray")
plot(byWeekEnd, type = "l", ylab = "Steps", xlab = "", main = "Weekend")
plot(byWeekday, type = "l", ylab = "Steps", xlab = "Time in 5 Minute Increments", 
    main = "Weekday")
```

![plot of chunk Chart4](figure/Chart4.png) 

