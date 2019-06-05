---
title: "Customer Segmentation"
author: "Árpád Tóth"
date: '2019 05 28 '
output: html_document
highlight: pygments
theme: spacelab
df_print: paged

---



## Import libraries


```r
library(data.table)
library(tidyverse)
```



## Importing 

I import directly the raw file from github.


```r
customers <- fread("https://raw.githubusercontent.com/SteffiPeTaffy/machineLearningAZ/master/Machine%20Learning%20A-Z%20Template%20Folder/Part%204%20-%20Clustering/Section%2025%20-%20Hierarchical%20Clustering/Mall_Customers.csv")
```

## Let!s explore the data


```r
glimpse(customers)
```

```
## Observations: 200
## Variables: 5
## $ CustomerID               <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, …
## $ Genre                    <chr> "Male", "Male", "Female", "Female", "Fe…
## $ Age                      <int> 19, 21, 20, 23, 31, 22, 35, 23, 64, 30,…
## $ `Annual Income (k$)`     <int> 15, 15, 16, 16, 17, 17, 18, 18, 19, 19,…
## $ `Spending Score (1-100)` <int> 39, 81, 6, 77, 40, 76, 6, 94, 3, 72, 14…
```

```r
summary(customers)
```

```
##    CustomerID        Genre                Age        Annual Income (k$)
##  Min.   :  1.00   Length:200         Min.   :18.00   Min.   : 15.00    
##  1st Qu.: 50.75   Class :character   1st Qu.:28.75   1st Qu.: 41.50    
##  Median :100.50   Mode  :character   Median :36.00   Median : 61.50    
##  Mean   :100.50                      Mean   :38.85   Mean   : 60.56    
##  3rd Qu.:150.25                      3rd Qu.:49.00   3rd Qu.: 78.00    
##  Max.   :200.00                      Max.   :70.00   Max.   :137.00    
##  Spending Score (1-100)
##  Min.   : 1.00         
##  1st Qu.:34.75         
##  Median :50.00         
##  Mean   :50.20         
##  3rd Qu.:73.00         
##  Max.   :99.00
```

```r
colSums(is.na(customers))
```

```
##             CustomerID                  Genre                    Age 
##                      0                      0                      0 
##     Annual Income (k$) Spending Score (1-100) 
##                      0                      0
```


There is no NA data. Cool! Then dive into clustering.

## Plot 3D 


```r
library(plotly)

customers$Genre <- as.factor(customers$Genre)

names(customers) <- c('id', 'genre', 'age', 'annual_income', 'spending_score')
customers <- customers %>% select(-id)

str(customers)
```

```
## Classes 'data.table' and 'data.frame':	200 obs. of  4 variables:
##  $ genre         : Factor w/ 2 levels "Female","Male": 2 2 1 1 1 1 1 1 2 1 ...
##  $ age           : int  19 21 20 23 31 22 35 23 64 30 ...
##  $ annual_income : int  15 15 16 16 17 17 18 18 19 19 ...
##  $ spending_score: int  39 81 6 77 40 76 6 94 3 72 ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

```r
p <- plot_ly(customers, x = ~age, y = ~annual_income, z = ~spending_score, color = ~genre, colors = c('#BF382A', '#0C4B8E')) %>%
  add_markers() %>%
  layout(scene = list(xaxis = list(title = 'age'),
                     yaxis = list(title = 'annual_income'),
                     zaxis = list(title = 'spending_score')))


p
```

```
## Error in loadNamespace(name): there is no package called 'webshot'
```


## Implementing K-means




```
## Error in loadNamespace(name): there is no package called 'webshot'
```

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.
