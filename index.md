---
title       : Dallas Data
subtitle    : How to reach similar results
author      : Jonathan
job         : 
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
---

## Slide 2

You have to clean the data initially

```r
library(xlsx)
## Downloading the data
fileUrl <- "http://raw.texastribune.org.s3.amazonaws.com/dallas/salaries/2015-04/cityofdallas0415.xls"
fileName <- "./data.xlsx"
download.file(fileUrl,fileName,mode="wb")
colClasses<-c("character","character","character","character","character","numeric","numeric","numeric")
dallasData<-read.xlsx2(fileName,sheetIndex=1,colClasses=colClasses,stringsAsFactors=FALSE) # So we can change the ethnicity later on based on clusters
dallasData <- as.data.frame(dallasData)

## Remove names and other data we will not use that take up space (make it nice and clean)
# Reduces file size from 2MB to 0.4MB
keep <- c("Gender", "Ethnicity", "Adjusted.Hire.Date", "Rate.of.Pay", "Annual.Hours")
dallasData <- dallasData[, (names(dallasData) %in% keep)]
```

--- .class #id

## Slide 3

After clustering by ethnicity (which is too lengthy to post in a slidify document), we clean up the Rate.of.Pay and re-write the data.

```r
## Re-Adjust the Rate.of.Pay variable in the data
# Some individuals have the Rate.of.Pay listed by the hourly earnings, and some have it listed by yearly earnings.
# This will standardize to make sure the data is consistent across all rows.
dallasData[dallasData$Rate.of.Pay < 81,]$Rate.of.Pay <- 
    dallasData[dallasData$Rate.of.Pay < 81,]$Rate.of.Pay * dallasData[dallasData$Rate.of.Pay < 81,]$Annual.Hours

## Change the names of the columns because javascript will have issues later on with periods in the column names
names(dallasData) <- c("Gender", "Ethnicity", "Adjusted_Hire_Date", "Rate_of_Pay", "Annual_Hours")

## Write the cleaned and processed data to an excel spreadsheet
write.xlsx2(dallasData, "CleanedDallasData.xlsx")
```

--- .class #id

## Slide 4

The ui.R is relatively straightforward.

```r
shinyUI(pageWithSidebar(
    headerPanel("Dallas Employment Data"),
    sidebarPanel(
        selectInput(inputId="eth2",
                    label="Select a list of ethnicities:",
                    choices=c("American Indian", "Asian", "Black", "Hispanic",
                              "Other Asian", "Pacific Islander", "White"),
                    multiple=TRUE),
        checkboxInput("divideByGender", "Divide by Gender", TRUE),
        checkboxInput("graphMeans", "Graph means instead of points", FALSE),
        
        h3("\n\n"),
        verbatimTextOutput("meanInformation")
    ),
    mainPanel(
        textOutput("\n"),
        showOutput("chart1", "polycharts") 
    )
))
```

```
## Error in eval(expr, envir, enclos): could not find function "shinyUI"
```

--- .class #id

## Slide 5

The majority of the computation was included in server.R


```r
options(
    RCHARD_LIB = "polycharts",
    rcharts.mode="iframesrc",
    rcharts.cdn=TRUE,
    RCHART_WIDTH=600,
    RCHART_HEIGHT=400
)
```

