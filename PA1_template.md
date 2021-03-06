# Reproducible Research: Peer Assessment 1


### Loading and preprocessing the data - te3st change 2

```r
#unzip and load the data into R
data <- read.csv(unz(filename = "activity.csv", description = "activity.zip"))
```

### What is mean total number of steps taken per day?


```r
library(ggplot2)
#summarise the data
summary <- aggregate(steps ~ date, data = data, sum)

#plot the data
ggplot(summary, aes(x = steps)) + 
  geom_histogram(binwidth = 5000, colour = "black", fill = "grey") +
  theme_bw() +
  geom_vline(aes(xintercept = mean(steps, na.rm = T)), colour = "red", linetype = "dashed", size = 1) +
  xlab("Total Daily Steps") +
  ylab("Frequency")
```

![plot of chunk generate_histogram](./PA1_template_files/figure-html/generate_histogram.png) 


```r
#turn off scientific notation
options("scipen" = 100, "digits" = 0)
#calculate mean and median values
steps_mean <- mean(summary$steps, na.rm = T)
steps_median <- median(summary$steps, na.rm = T)
```

1. The **mean** total number of steps taken each day is 10,766  
2. The **median** total number of steps taken each day is 10,765

### What is the average daily activity pattern?
####Part 1

```r
 #aggregate the data by interval 
 summary_interval <- aggregate(steps ~ interval, data = data, mean)

 #plot the data
 ggplot(data = summary_interval, aes(x = interval, y = steps, group = 1)) + 
  geom_line() +
  xlab("Time") +
  ylab("Average Number Of Steps") +
  theme_bw()
```

![plot of chunk time_series](./PA1_template_files/figure-html/time_series.png) 

####Part 2


```r
#find the internval with the highest average number of steps
top_interval <- summary_interval[summary_interval$steps == max(summary_interval$steps),1]
```

The time inverval is the **highest average number of steps** is 835


### Imputing missing values

1. Number of NAs

```r
#total NAs in steps colum
total_na <- sum(is.na(data$steps))
```

The **total** number of NAs is 2304

2. Replace NAs
3. Create a clean dataset

```r
#inserting missing values
#using the ddply() function from plyr package, replace NAs with the mean value 
#for the interval
library(plyr)
#fuction to replace NAs with the mean value
impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))
data.clean <- ddply(data, .(interval), transform, steps = impute.mean(steps))
```

4. Histogram with clean data
a. Histogram

```r
#get the total step per day for the clean dataset
total_day <- ddply(data.clean, .(date), function(x) sum(x$steps))
colnames(total_day)[2] <- "steps"

#plot the graph
ggplot(total_day, aes(x = steps)) +
  geom_histogram(binwidth = 5000, colour = "black", fill = "grey") +
   theme_bw() +
  geom_vline(aes(xintercept = mean(steps, na.rm = T)), colour = "red", linetype = "dashed", size = 1) +
  xlab("Total Daily Steps") +
  ylab("Frequency")
```

![plot of chunk histogram](./PA1_template_files/figure-html/histogram.png) 

b. Mean and Median total steps per day  


```r
#turn off scientific notation
options("scipen" = 100, "digits" = 0)
#calculate mean and median values
steps_mean <- mean(total_day$steps)
steps_median <- median(total_day$steps)
```

1. The **mean** total number of steps taken each day is 10,766  
2. The **median** total number of steps taken each day is 10,766

c. Difference between data with and without NAs  


```r
#create an extra field in the 2 datasets indicating whether NA has been included or changed
#Yes = NAs have been included
#No = NAs have been changed to the mean value
summary <- data.frame(summary, NA_inc = as.character("Yes"))
total_day <- data.frame(total_day, NA_inc = as.character("No"))

#combine the 2 datasets
all.data <- rbind(summary,total_day)

#create a box plot to show the difference between the 2 datasets
ggplot(all.data, aes(x = NA_inc, y = steps, fill = NA_inc )) + 
  geom_boxplot() +
  guides(fill = F) + #remove legend
  coord_flip() +
  ylab("Total Steps") +
  xlab("NA Included")
```

![plot of chunk compare_boxplot](./PA1_template_files/figure-html/compare_boxplot.png) 


```r
#histograms to compare the 2 datasets side by side
ggplot(all.data, aes(x = steps)) + 
  geom_histogram(binwidth = 5000, colour = "black", fill = "white") +
  facet_grid(. ~ NA_inc) +
  theme_bw() +
  geom_vline(aes(xintercept = mean(steps, na.rm = T)), colour = "red", linetype = "dashed", size = 1) +
  xlab("Total Daily Steps") +
  ylab("Frequency")
```

![plot of chunk compare_histogram](./PA1_template_files/figure-html/compare_histogram.png) 

d. Imact  
We can see that the mean and median are the same. From the box plot we can see that when we remove NAs and replace them with the mean value for each interval, the IQR get smaller and we get a few more outliers.
From the side-by-side histogram we see that the overall shape has not changed but the frequency of the highest bin range has increased.

### Are there differences in activity patterns between weekdays and weekends?


```r
#covert data to POSIXlt
data.clean$date <- as.POSIXlt(data.clean$date)

options(scipen=999)
#create factor variable that identifies whether a date is weekend(1) or not(0)
data.clean <- data.frame(data.clean, isweekend = ifelse(weekdays(data.clean$date, abbreviate = T) == "Sun" | weekdays(data.clean$date,abbreviate = T) == "Sat","Weekend","Weekday"))


weekend_summary <- ddply(data.clean, c("interval","isweekend"), summarise, mean = mean(steps))
```


```r
ggplot(data = weekend_summary, aes(x = interval, y = mean, colour = isweekend)) +
  geom_line() +
  facet_grid(isweekend ~ .) +
  theme_bw() +
  theme(legend.position="none") +
  xlab("Time Interval") +
  ylab("Average Steps")
```

![plot of chunk draw_line_graph](./PA1_template_files/figure-html/draw_line_graph.png) 

#####Analysis:  
From the 2 plots we can see that during the weekdays most of the walking happens betweeen 5:30am and 8:30am. Very little is done during the evening period from 8pm onwards. 
On the other hand, during the weekend, the walking is more spread out, there is not as much fluctuation throughout the day. Most of the walking is done from about 8:30am, very little is done before this time. Lastly, more walking is done in the evenining from about 8pm then during the weekdays.
