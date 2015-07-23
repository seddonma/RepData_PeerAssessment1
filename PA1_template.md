----
title: 'Reproducible Research - Assignment #2'
author: "seddonma"
date: "July 25, 2015"
---

# Population and Economic Costs of Severe Weather Events

## Synopsis 

Storms and other severe weather events can cause both public health and economic problems for communities and municipalities. Many severe events can result in fatalities, injuries, and property damage, and preventing such outcomes to the extent possible is a key concern. This project involves exploring the U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database. This database tracks characteristics of major storms and weather events in the United States, including when and where they occur, as well as estimates of any fatalities, injuries, and property damage.

The research questions for this study are as follows:

1.  Across the United States, which types of events (as indicated in the EVTYPE variable) are most harmful with       respect to population health?

2.  Across the United States, which types of events have the greatest economic consequences?

## Data Processing    
    
The data was most recently downloaded in July 25th, 2015. The file downloaded is a comma-separated-value file compressed via the bzip2 algorithm. 
```{r}
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
destination <- "repdata-data-StormData.csv.bz2"
setInternet2(use = TRUE)
download.file(url, destination)
dataset <- read.csv("repdata-data-StormData.csv.bz2", sep = ",", header = TRUE)
```

This data analysis requires the use of the plyr and ggplot2 packages. The following code snippet loads the two packages.
```{r}
library(plyr)
library(ggplot2)
```

```{r}
str(dataset)
```
Using the dataset's documentation, an initial look at the full dataset shows a large number of variables that are irrelevant to the research question, such as the location and time that each event occurred. Therefore these variables are removed. In addition, in order to optimize cleaning and analyzing the dataset, observations that do not list any population or economic damage are removed.
```{r}
dataset <- dataset[,c("EVTYPE", "FATALITIES", "INJURIES", "PROPDMG", "PROPDMGEXP", "CROPDMG", "CROPDMGEXP")]
dataset <- subset(dataset, FATALITIES != 0 | INJURIES != 0 | PROPDMG != 0 | CROPDMG != 0)
```

In order to measure the impact of a weather event on the population's health, the number of injuries and fatalities that directly result from the weather event are summed into a variable called "POPDMG" 
```{r}
dataset$POPDMG <- dataset$INJURIES + dataset$FATALITIES
```

In order to determine the total economic cost of each weather event, many transformations needed to be made. 
```{r}
dataset <- dataset[!(dataset$PROPDMGEXP %in% c("-", "+", "?", "0", "2", "3", "4", "5", "6", "7", "8")),]
dataset <- dataset[!(dataset$CROPDMGEXP %in% c("?", "0")),]
dataset$PROPDMGEXP <- toupper(dataset$PROPDMGEXP)
dataset$CROPDMGEXP <- toupper(dataset$CROPDMGEXP)
dataset$PROPDMGEXP[dataset$PROPDMGEXP == "H"] <- 100 
dataset$PROPDMGEXP[dataset$PROPDMGEXP == "K"] <- 1000
dataset$PROPDMGEXP[dataset$PROPDMGEXP == "M"] <- 1000000
dataset$PROPDMGEXP[dataset$PROPDMGEXP == "B"] <- 1000000000
dataset$CROPDMGEXP[dataset$CROPDMGEXP == "H"] <- 100 
dataset$CROPDMGEXP[dataset$CROPDMGEXP == "K"] <- 1000
dataset$CROPDMGEXP[dataset$CROPDMGEXP == "M"] <- 1000000
dataset$CROPDMGEXP[dataset$CROPDMGEXP == "B"] <- 1000000000
dataset$PROPDMGEXP <- sub("^$", 0, dataset$PROPDMGEXP)
dataset$CROPDMGEXP <- sub("^$", 0, dataset$CROPDMGEXP)
dataset$ECODMG <- (dataset$PROPDMG*as.numeric(dataset$PROPDMGEXP)) + (dataset$CROPDMG*as.numeric(dataset$CROPDMGEXP))
```
## Results

There should be a section titled Results in which your results are presented.


The analysis document must have at least one figure containing a plot.Your analysis must have no more than three figures. Figures may have multiple plots in them (i.e. panel plots), but there cannot be more than three figures total.

You must show all your code for the work in your analysis document. This may make the document a bit verbose, but that is okay. In general, you should ensure that echo = TRUE for every code chunk (this is the default setting in knitr).
