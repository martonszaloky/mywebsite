---
categories:
- ""
- ""
date: "2017-10-31T22:26:13-05:00"
description: Tracking GDP component changes
draft: false
image: pic08.jpg
keywords: ""
slug: tempus
title: GDP Components
---


```{r, setup, echo=FALSE}
knitr::opts_chunk$set(
  message = FALSE, 
  warning = FALSE, 
  tidy=FALSE,     # display code as typed
  size="small")   # slightly smaller font for code
options(digits = 3)

# default figure size
knitr::opts_chunk$set(
  fig.width=6.75, 
  fig.height=6.75,
  fig.align = "center"
)
```


```{r load-libraries, echo=FALSE}
library(tidyverse)  # Load ggplot2, dplyr, and all the other tidyverse packages
library(mosaic)
library(ggthemes)
library(GGally)
library(readxl)
library(here)
library(skimr)
library(janitor)
library(broom)
library(tidyquant)
library(infer)
library(openintro)
library(tidyquant)
library(infer)
```


# Challenge 2:GDP components over time and among countries

At the risk of oversimplifying things, the main components of gross domestic product, GDP are personal consumption (C), business investment (I), government spending (G) and net exports (exports - imports). You can read more about GDP and the different approaches in calculating at the [Wikipedia GDP page](https://en.wikipedia.org/wiki/Gross_domestic_product).

The GDP data we will look at is from the [United Nations' National Accounts Main Aggregates Database](https://unstats.un.org/unsd/snaama/Downloads), which contains estimates of total GDP and its components for all countries from 1970 to today. We will look at how GDP and its components have changed over time, and compare different countries and how much each component contributes to that country's GDP. The file we will work with is [GDP and its breakdown at constant 2010 prices in US Dollars](http://unstats.un.org/unsd/amaapi/api/file/6) and it has already been saved in the Data directory. Have a look at the Excel file to see how it is structured and organised


```{r read_GDP_data}

UN_GDP_data  <-  read_excel(here::here("data", "Download-GDPconstant-USD-countries.xls"), # Excel filename
                sheet="Download-GDPconstant-USD-countr", # Sheet name
                skip=2) # Number of rows to skip

```

 The first thing you need to do is to tidy the data, as it is in wide format and you must make it into long, tidy format. Please express all figures in billions (divide values by `1e9`, or $10^9$), and you want to rename the indicators into something shorter.

> make sure you remove `eval=FALSE` from the next chunk of R code-- I have it there so I could knit the document

```{r reshape_GDP_data}
UN_GDP_data$IndicatorName %>% unique()

tidy_GDP_data  <- UN_GDP_data %>% 
  pivot_longer(
    cols="1970":"2017",
    names_to = "year",  
    values_to= "value" 
  ) %>% 
  mutate(
    year= year(as.Date(year, "%Y")),
    value = value/(10^9),
    IndicatorName= case_when(
           IndicatorName == "General government final consumption expenditure" ~ "Government expenditure",
           IndicatorName == "Household consumption expenditure (including Non-profit institutions serving households)" ~ "Household expenditure",
           IndicatorName =="Gross capital formation"~"Gross capital formation",
           IndicatorName == "Imports of goods and services" ~ "Imports",
           IndicatorName == "Exports of goods and services" ~ "Exports",
           IndicatorName == "Gross fixed capital formation (including Acquisitions less disposals of valuables)" ~ "Gross fixed cap formation",
           IndicatorName == "Gross Domestic Product (GDP)" ~ "GDP",
           
           # Not shortening the indicator names that are already short
           TRUE ~ IndicatorName
           
         )
  )


glimpse(tidy_GDP_data)


# Let us compare GDP components for these 3 countries
country_list <- c("United States","India", "Germany")
```