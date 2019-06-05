---
title: "Customer Segmentation"
author: "Árpád Tóth"
date: '2019 05 28 '
output: 
  html_document:
    keep_md: true
highlight: pygments
theme: spacelab
df_print: paged

---



## Import libraries


```r
library(data.table)
library(tidyverse)
```

```
## Registered S3 methods overwritten by 'ggplot2':
##   method         from 
##   [.quosures     rlang
##   c.quosures     rlang
##   print.quosures rlang
```

```
## ── Attaching packages ────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ ggplot2 3.1.1     ✔ purrr   0.3.2
## ✔ tibble  2.1.1     ✔ dplyr   0.8.1
## ✔ tidyr   0.8.3     ✔ stringr 1.4.0
## ✔ readr   1.3.1     ✔ forcats 0.4.0
```

```
## ── Conflicts ───────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::between()   masks data.table::between()
## ✖ dplyr::filter()    masks stats::filter()
## ✖ dplyr::first()     masks data.table::first()
## ✖ dplyr::lag()       masks stats::lag()
## ✖ dplyr::last()      masks data.table::last()
## ✖ purrr::transpose() masks data.table::transpose()
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
```

```
## 
## Attaching package: 'plotly'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     last_plot
```

```
## The following object is masked from 'package:stats':
## 
##     filter
```

```
## The following object is masked from 'package:graphics':
## 
##     layout
```

```r
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

<!--html_preserve--><div id="htmlwidget-f3b27010176cecaed872" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-f3b27010176cecaed872">{"x":{"visdat":{"d14a158da5d7":["function () ","plotlyVisDat"]},"cur_data":"d14a158da5d7","attrs":{"d14a158da5d7":{"x":{},"y":{},"z":{},"color":{},"colors":["#BF382A","#0C4B8E"],"alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"type":"scatter3d","mode":"markers","inherit":true}},"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"scene":{"xaxis":{"title":"age"},"yaxis":{"title":"annual_income"},"zaxis":{"title":"spending_score"}},"hovermode":"closest","showlegend":true},"source":"A","config":{"showSendToCloud":false},"data":[{"x":[20,23,31,22,35,23,30,35,58,24,35,35,46,54,45,40,23,21,49,21,42,30,36,20,65,31,49,24,50,27,29,31,49,31,50,51,27,67,54,43,68,32,47,60,60,45,23,49,46,21,55,22,34,50,68,40,32,47,27,23,49,21,66,19,38,18,19,63,49,51,50,38,40,23,31,25,31,29,44,35,57,28,32,32,34,44,38,47,27,30,30,56,29,31,36,33,36,52,30,37,32,29,41,54,41,36,34,32,38,47,35,45],"y":[16,16,17,17,18,18,19,19,20,20,21,23,25,28,28,29,29,30,33,33,34,34,37,37,38,39,39,39,40,40,40,40,42,43,43,44,46,47,47,48,48,48,49,50,50,54,54,54,54,54,57,57,58,58,59,60,60,60,60,62,62,62,63,63,64,65,65,65,65,67,67,67,69,70,70,72,72,73,73,74,75,76,76,77,78,78,78,78,78,78,78,79,79,81,85,86,87,88,88,97,97,98,99,101,103,103,103,103,113,120,120,126],"z":[6,77,40,76,6,94,72,99,15,77,35,98,5,14,32,31,87,73,14,81,17,73,26,75,35,61,28,65,55,47,42,42,52,54,45,50,51,52,59,50,48,47,42,49,56,53,52,42,44,57,58,55,60,46,55,40,42,47,50,41,48,42,50,54,42,48,50,43,59,43,57,40,58,29,77,34,71,88,7,72,5,40,87,74,22,20,76,16,89,78,73,35,83,93,75,95,27,13,86,32,86,88,39,24,17,85,23,69,91,16,79,28],"type":"scatter3d","mode":"markers","name":"Female","marker":{"color":"rgba(191,56,42,1)","line":{"color":"rgba(191,56,42,1)"}},"textfont":{"color":"rgba(191,56,42,1)"},"error_y":{"color":"rgba(191,56,42,1)"},"error_x":{"color":"rgba(191,56,42,1)"},"line":{"color":"rgba(191,56,42,1)"},"frame":null},{"x":[19,21,64,67,37,22,20,52,35,25,31,29,35,60,53,18,24,48,33,59,47,69,53,70,19,63,18,19,70,59,26,40,57,38,67,48,18,48,24,48,20,67,26,49,54,68,66,65,19,27,39,43,40,59,38,47,39,20,32,19,32,25,28,48,34,43,39,37,34,19,50,42,32,40,28,36,36,58,27,59,35,46,30,28,33,32,32,30],"y":[15,15,19,19,20,20,21,23,24,24,25,28,28,30,33,33,38,39,42,43,43,44,46,46,46,48,48,48,49,54,54,54,54,54,54,54,59,60,60,61,61,62,62,62,63,63,63,63,64,67,69,71,71,71,71,71,71,73,73,74,75,77,77,77,78,78,78,78,78,81,85,86,87,87,87,87,87,88,88,93,93,98,99,101,113,126,137,137],"z":[39,81,3,14,13,79,66,29,35,73,73,82,61,4,4,92,92,36,60,60,41,46,46,56,55,51,59,59,55,47,54,48,51,55,41,46,41,49,52,42,49,59,55,56,46,43,48,52,46,56,91,35,95,11,75,9,75,5,73,10,93,12,97,36,90,17,88,1,1,5,26,20,63,13,75,10,92,15,69,14,90,15,97,68,8,74,18,83],"type":"scatter3d","mode":"markers","name":"Male","marker":{"color":"rgba(12,75,142,1)","line":{"color":"rgba(12,75,142,1)"}},"textfont":{"color":"rgba(12,75,142,1)"},"error_y":{"color":"rgba(12,75,142,1)"},"error_x":{"color":"rgba(12,75,142,1)"},"line":{"color":"rgba(12,75,142,1)"},"frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


## Implementing K-means



<!--html_preserve--><div id="htmlwidget-7b46e0993b549b584405" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-7b46e0993b549b584405">{"x":{"visdat":{"d14a191d8768":["function () ","plotlyVisDat"]},"cur_data":"d14a191d8768","attrs":{"d14a191d8768":{"x":{},"y":{},"z":{},"color":{},"colors":["#BF382A","#0C4B8E"],"alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"type":"scatter3d","mode":"markers","inherit":true}},"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"scene":{"xaxis":{"title":"age"},"yaxis":{"title":"annual_income"},"zaxis":{"title":"spending_score"}},"hovermode":"closest","showlegend":true},"source":"A","config":{"showSendToCloud":false},"data":[{"x":[39,31,40,38,39,31,29,32,35,32,32,28,32,34,39,38,27,30,30,29,31,36,33,32,28,36,30,27,35,32,29,30,28,36,32,38,35,32,30],"y":[69,70,71,71,71,72,73,73,74,75,76,77,77,78,78,78,78,78,78,79,81,85,86,87,87,87,88,88,93,97,98,99,101,103,103,113,120,126,137],"z":[91,77,95,75,75,71,88,73,72,93,87,97,74,90,88,76,89,78,73,83,93,75,95,63,75,92,86,69,90,86,88,97,68,85,69,91,79,74,83],"type":"scatter3d","mode":"markers","name":"1","marker":{"color":"rgba(191,56,42,1)","line":{"color":"rgba(191,56,42,1)"}},"textfont":{"color":"rgba(191,56,42,1)"},"error_y":{"color":"rgba(191,56,42,1)"},"error_x":{"color":"rgba(191,56,42,1)"},"line":{"color":"rgba(191,56,42,1)"},"frame":null},{"x":[23,59,47,25,20,44,19,57,25,48,34,43,44,47,37,34,56,19,50,42,36,40,36,52,58,59,37,46,41,54,41,34,33,47,45,32],"y":[70,71,71,72,73,73,74,75,77,77,78,78,78,78,78,78,79,81,85,86,87,87,87,88,88,93,97,98,99,101,103,103,113,120,126,137],"z":[29,11,9,34,5,7,10,5,12,36,22,17,20,16,1,1,35,5,26,20,27,13,10,13,15,14,32,15,39,24,17,23,8,16,28,18],"type":"scatter3d","mode":"markers","name":"2","marker":{"color":"rgba(164,64,68,1)","line":{"color":"rgba(164,64,68,1)"}},"textfont":{"color":"rgba(164,64,68,1)"},"error_y":{"color":"rgba(164,64,68,1)"},"error_x":{"color":"rgba(164,64,68,1)"},"line":{"color":"rgba(164,64,68,1)"},"frame":null},{"x":[19,20,31,35,64,67,58,37,35,52,35,46,54,45,40,60,53,49,42,36,65,48,49],"y":[15,16,17,18,19,19,20,20,21,23,24,25,28,28,29,30,33,33,34,37,38,39,39],"z":[39,6,40,6,3,14,15,13,35,29,35,5,14,32,31,4,4,14,17,26,35,36,28],"type":"scatter3d","mode":"markers","name":"3","marker":{"color":"rgba(134,69,93,1)","line":{"color":"rgba(134,69,93,1)"}},"textfont":{"color":"rgba(134,69,93,1)"},"error_y":{"color":"rgba(134,69,93,1)"},"error_x":{"color":"rgba(134,69,93,1)"},"line":{"color":"rgba(134,69,93,1)"},"frame":null},{"x":[50,27,29,31,49,33,31,59,50,47,51,69,27,53,70,19,67,54,63,18,43,68,19,32,70,47,60,60,59,26,45,40,23,49,57,38,67,46,21,48,55,22,34,50,68,18,48,40,32,24,47,27,48,20,23,49,67,26,49,21,66,54,68,66,65,19,38,19,18,19,63,49,51,50,27,38,40,43,28],"y":[40,40,40,40,42,42,43,43,43,43,44,44,46,46,46,46,47,47,48,48,48,48,48,48,49,49,50,50,54,54,54,54,54,54,54,54,54,54,54,54,57,57,58,58,59,59,60,60,60,60,60,60,61,61,62,62,62,62,62,62,63,63,63,63,63,63,64,64,65,65,65,65,67,67,67,67,69,71,76],"z":[55,47,42,42,52,60,54,60,45,41,50,46,51,46,56,55,52,59,51,59,50,48,59,47,55,42,49,56,47,54,53,48,52,42,51,55,41,44,57,46,58,55,60,46,55,41,49,40,42,52,47,50,42,49,41,48,59,55,56,42,50,46,43,48,52,54,42,46,48,50,43,59,43,57,56,40,58,35,40],"type":"scatter3d","mode":"markers","name":"4","marker":{"color":"rgba(96,73,117,1)","line":{"color":"rgba(96,73,117,1)"}},"textfont":{"color":"rgba(96,73,117,1)"},"error_y":{"color":"rgba(96,73,117,1)"},"error_x":{"color":"rgba(96,73,117,1)"},"line":{"color":"rgba(96,73,117,1)"},"frame":null},{"x":[21,23,22,23,30,35,24,22,20,35,25,31,29,35,23,21,18,21,30,20,24,31,24],"y":[15,16,17,18,19,19,20,20,21,23,24,25,28,28,29,30,33,33,34,37,38,39,39],"z":[81,77,76,94,72,99,77,79,66,98,73,73,82,61,87,73,92,81,73,75,92,61,65],"type":"scatter3d","mode":"markers","name":"5","marker":{"color":"rgba(12,75,142,1)","line":{"color":"rgba(12,75,142,1)"}},"textfont":{"color":"rgba(12,75,142,1)"},"error_y":{"color":"rgba(12,75,142,1)"},"error_x":{"color":"rgba(12,75,142,1)"},"line":{"color":"rgba(12,75,142,1)"},"frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.
