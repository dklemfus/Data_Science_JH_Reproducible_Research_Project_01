---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


### Loading and preprocessing the data




```r
# Required Packages: 
# dplyr
# data.table
# ggplot2

# Load compressed data:
raw.data <- read.csv(unzip('./activity.zip'))

# Process data by converting date to POSIX:
processed.data <- raw.data %>% mutate(
  interval = sprintf('%04d',interval), # Pad leading zeros
  dateTime = as.POSIXct(paste0(date,' ',interval), format="%Y-%m-%d %H%M"),
  date=as.Date(date, format="%Y-%m-%d"),
  )
```

### What is mean total number of steps taken per day?

```r
# Create clean data with NA values removed:
step.data <- processed.data %>% filter(!is.na(steps)) %>% group_by(date)

# Determine the Total Steps Taken Per Day:
total.steps <- step.data %>% summarize(Count = sum(steps))

# Determine Mean/Median of steps per day: 
mean.steps <- round(mean(total.steps$Count, na.rm=T),0)
median.steps <- round(median(total.steps$Count, na.rm=T),0)

# Create Histogram showing total number of steps per day: 
hist(total.steps$Count, xlab = "Steps Taken Per Day", 
     main="Histogram: Steps Taken Per Day")
```

![](PA1_template_files/figure-html/mean_steps-1.png)<!-- -->

```r
cat(paste0("Mean Total Number of Steps Per Day: ", mean.steps))
```

```
## Mean Total Number of Steps Per Day: 10766
```

```r
cat(paste0("Median Total Number of Steps Per Day: ", median.steps))
```

```
## Median Total Number of Steps Per Day: 10765
```

### What is the average daily activity pattern?

```r
# Determine the average steps per 5-minute interval over data set: 
daily.data <- processed.data %>% filter(!is.na(steps)) %>% group_by(interval) %>%
  summarize(average_steps = mean(steps))

# Plot the Time Series Plot on 5-minute interval for average steps taken:
ggplot(daily.data) +
  geom_line(aes(x=factor(interval), y=average_steps, group=1)) +
  scale_x_discrete(breaks=daily.data$interval[c(T,rep(F,times=29))]) +
  theme(axis.text.x = element_text(angle = -90, hjust = 1),
        axis.title.x = element_blank()) 
```

![](PA1_template_files/figure-html/daily_activity-1.png)<!-- -->

```r
# Determine Which 5-minute interval contains maximum number of steps: 
cat(paste0("Interval (5-min) Containing Max Steps: ", 
           daily.data$interval[which.max(daily.data$average_steps)]))
```

```
## Interval (5-min) Containing Max Steps: 0835
```


### Imputing missing values

```r
# Determine number of missing values: 
missing <- sum(is.na(processed.data$steps)) # Note: Other fields didn't contain NA
cat(paste0("Total Number of Missing Values (NA): ", missing))
```

```
## Total Number of Missing Values (NA): 2304
```

```r
# Strategy for filling in NA's: Use Mean value for 5-minute interval, since in 
# certain cases the entire day is missing, making it difficult to interpolate

cat("To approximate missing values, use the mean value for the 5-minute interval")
```

```
## To approximate missing values, use the mean value for the 5-minute interval
```

```r
# Create new dataset using strategy: 
new.processed.data <- processed.data %>% mutate(
  steps = ifelse(!is.na(steps),steps, # If not NA, keep original value
                  daily.data$average_steps[chmatch(interval, daily.data$interval)])
)

# Create new histogram of data showing steps per day after correction: 
new.total.steps <- new.processed.data %>% group_by(date) %>% summarize(Count = sum(steps))

# Determine Mean/Median of steps per day after correction: 
new.mean.steps <- round(mean(new.total.steps$Count),0)
new.median.steps <- round(median(new.total.steps$Count),0)

hist(new.total.steps$Count, xlab = "Steps Taken Per Day (Including Missing Correction)", 
     main="Histogram: Steps Taken Per Day")
```

![](PA1_template_files/figure-html/missing_values-1.png)<!-- -->

```r
cat(paste0("Mean Total Number of Steps Per Day (corrected): ", new.mean.steps))
```

```
## Mean Total Number of Steps Per Day (corrected): 10766
```

```r
cat(paste0("Median Total Number of Steps Per Day (corrected): ", new.median.steps))
```

```
## Median Total Number of Steps Per Day (corrected): 10766
```

```r
# Determine the impact of inputting missing data on total daily steps: 
if (sum(new.total.steps$Count) > sum(total.steps$Count)){
  cat(paste0("Inputting Missing Data INCREASED total number of daily steps: "),
      "(",sum(new.total.steps$Count)," vs. ", sum(total.steps$Count),")")
} else if (sum(new.total.steps$Count) < sum(total.steps$Count)){
    cat(paste0("Inputting Missing Data DECREASED total number of daily steps: "),
        "(",sum(new.total.steps$Count)," vs. ", sum(total.steps$Count),")")
} else {
  cat("Inputting Missing Data had no change on total number of daily steps")
}
```

```
## Inputting Missing Data INCREASED total number of daily steps:  ( 656737.5  vs.  570608 )
```

### Are there differences in activity patterns between weekdays and weekends?

```r
new.processed.data <- new.processed.data %>% mutate(
  weekday = weekdays(date),
  w = case_when(grepl('Saturday|Sunday',weekday) ~ "Weekend",
                TRUE ~ "Weekday")
)

# Render Plot of weekday vs. weekends
ggplot(new.processed.data, aes(x=interval, y=steps)) +
  geom_line() +
  scale_x_discrete(breaks=new.processed.data$interval[c(T,rep(F,time=29))]) +
  facet_wrap(~w, ncol=1) +
  theme(axis.text.x = element_text(angle = -90, hjust = 1),
        axis.title.x = element_blank())
```

![](PA1_template_files/figure-html/weekday_weekend-1.png)<!-- -->


