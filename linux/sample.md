my Raspberry Pi Stats
========================================================

This is a sample document showing what will be done in the finished document itself. It's just for **testing**.


```r
library(RSQLite)
```

```
## Loading required package: methods
```

```
## Loading required package: DBI
```

```r
library(ggplot2)
library(reshape)
```

```
## Loading required package: plyr
```

```
## Attaching package: 'reshape'
```

```
## The following object(s) are masked from 'package:plyr':
## 
## rename, round_any
```

```r


# db-Connection
con <- dbConnect("SQLite", dbname = "data/data.db")

# cron-time (minutes)
ct <- 5

# load data from yesterday data <- dbGetQuery(con, 'select * from data;')
data <- dbGetQuery(con, "select * from data where date(date) == date('now', '-1 days');")

# data <- read.csv('data/sample_data.txt', sep = ';') load als numeric
# data$load1 <- as.numeric(levels(data$load1))[data$load1] data$load5 <-
# as.numeric(levels(data$load5))[data$load5] data$load15 <-
# as.numeric(levels(data$load15))[data$load15]

# RAM in Megabytes
data$mem_tot <- round(data$mem_tot/1024, digits = 1)
data$mem_free <- round(data$mem_free/1024, digits = 1)
data$mem_buffers <- round(data$mem_buffers/1024, digits = 1)
data$mem_cached <- round(data$mem_cached/1024, digits = 1)
data$mem_div <- round(data$mem_div/1024, digits = 1)

# network activity in Kilobytes
data$rx_kbytes <- round(data$rx_bytes/1024)
data$tx_kbytes <- round(data$tx_bytes/1024)

# calculate the network-activity between two timepoints some kernel still
# reset the counter to 0 if there are more than 4GB (4194304 KB)
# transferred, so we have to recalculate this
data$diff_rx <- c(0, diff(data$rx_kbytes))/(ct * 60)
data$diff_tx <- c(0, diff(data$tx_kbytes))/(ct * 60)

for (i in 2:nrow(data)) {
    if (data$diff_rx[i] < 0) {
        data$diff_rx[i] <- data$diff_rx[i] + (4 * 1024 * 1024/(ct * 60))
    }
    if (data$diff_tx[i] < 0) {
        data$diff_tx[i] <- data$diff_tx[i] + (4 * 1024 * 1024/(ct * 60))
    }
}

# format date
data$date <- as.POSIXlt(data$date, format = "%Y-%m-%d %H:%M:%S")
```


CPU load
--------


```r
temp_data <- melt(data[, c(1, 2, 3, 4)], id = c("date"))

load_plot <- ggplot(temp_data, aes(x = date, y = value, group = variable)) + 
    geom_line(aes(colour = variable))
# load_plot <- ggplot(temp_data, aes(x = date_new, y = value, group =
# variable)) + geom_smooth(aes(colour = variable))
load_plot
```

![plot of chunk cpu-daily](figure/cpu-daily.png) 

```r
# plot(data$time, data$load1) lines(data$time, data$load1)
```



Temperature
-----------


```r
temp_plot <- ggplot(data, aes(x = date, y = temp, width = nrow(data) + 20)) + 
    geom_bar(stat = "identity")
temp_plot
```

```
## Warning: position_stack requires non-overlapping x intervals
```

![plot of chunk temp-daily](figure/temp-daily.png) 



Memory Usage
------------


```r
temp_data <- melt(data[, c(1, 9, 10, 11, 12)], id = c("date"))

mem_plot <- ggplot(temp_data, aes(x = date, y = value, fill = variable, width = nrow(data) + 
    20)) + geom_bar(stat = "identity")
mem_plot
```

```
## Warning: position_stack requires non-overlapping x intervals
```

![plot of chunk mem-daily](figure/mem-daily.png) 



Network
-------


```r
temp_data <- melt(data[, c(1, 15, 16)], id = c("date"))

net_plot <- ggplot(temp_data, aes(x = date, y = value, group = variable)) + 
    geom_line(aes(colour = variable))
net_plot <- net_plot + ylab("kbytes/s") + xlab("time") + scale_y_sqrt()
net_plot
```

![plot of chunk net-daily](figure/net-daily.png) 

```r
# plot(data$time, data$load1) lines(data$time, data$load1)
```
