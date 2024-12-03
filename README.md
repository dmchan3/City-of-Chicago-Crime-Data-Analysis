# Analysis of Chicago Crime 2020-2024

## Research Topic
This project will analyze City of Chicago crime data to explore patterns and insights over the past 5 years. Using this large dataset (200-300k records per year), our aim is to identify trends and examine crime intensity based on locations, times, and types of offenses. Specifically, we will explore questions such as which blocks or streets are the most dangerous, seasonal crime patterns, and how these trends have evolved.

## Team Members
- David Chan
- Croix Westbrock
- Srishti Nandal 
- Matthew Ritland

## Data

### Description of the Dataset
Our primary dataset comes from the City of Chicago government open data portal, containing records of reported crimes. The link to the full dataset (2001-Present) is https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2/about_data . For this project, we are using a subset of this dataset to focus on the last 5 years. CSV files for specific years, as we will be using, can be found on the site linked above by searching "Crimes - *Year*".

### Questions to be Addressed
* What are the most committed crimes in Chicago? Which crimes lead to the most arrests? How many crimes are domestic versus non-domestic?
* Are there any seasonal crime patterns? Does quantity or types of crimes vary noticeably with changing of months and seasons?
* How was Chicago crime affected by COVID-19? Is there a noticeable difference in the quantity and/or types of crimes committed during COVID-19 times versus recent "normal" times?
* What areas of Chicago produce the most crimes? Block? District?
* O Block is widely known as one of the most dangerous blocks in Chicago and the United States as a whole. How do O Block crime rates compare to the rest of Chicago?
* What are the most dangerous location types? Is there any change by year?
* What areas of Chicago hold the most arrests? Does this differ from the whole dataset?

### Initial Data Cleaning Steps

``` r
library(tidyverse)
```

While trying to load the datasets for each year, we ran into a problem where each of the files were too large for Github's 25mb file size limit. To work around this, we worked in a separate R script file to first eliminate the columns that were repetitive or not useful to our project. This narrowed our data to 12 columns from the previous 22. We then split each csv file into 2 different csv files, with the first including the first 4 columns, and the second containing the last 8. This split made each file similar in size, and more importantly, under the Github file size threshold. The code for this process for the 2024 dataset is given below as an example (with everything commented out as it will not run here)

``` r
# data2024 <- read.csv("ProjectFiles/Crimes - 2024.csv")
# c1_2024 <- data2024 %>% select(Date, Block, Primary.Type, Description)
# c2_2024 <- data2024 %>% select(Location.Description, Arrest, Domestic, District, Ward, Year, Latitude, Longitude)
# write.csv(c1_2024, file = "./ProjectFiles/2024_1.csv")
# write.csv(c2_2024, file = "./ProjectFiles/2024_2.csv")
```

We then merged the 2 subfiles back into one single year file to be used here.

``` r
crimes_2020 <- merge(read.csv("2020_1.csv"), read.csv("2020_2.csv"))
crimes_2021 <- merge(read.csv("2021_1.csv"), read.csv("2021_2.csv"))
crimes_2022 <- merge(read.csv("2022_1.csv"), read.csv("2022_2.csv"))
crimes_2023 <- merge(read.csv("2023_1.csv"), read.csv("2023_2.csv"))
crimes_2024 <- merge(read.csv("2024_1.csv"), read.csv("2024_2.csv"))
```

We then needed to combine all years into one main dataframe for analysis and remove the `X` column, which acted as the row number for records in each individual year dataset.

``` r
crimes <- rbind(crimes_2020, crimes_2021, crimes_2022, crimes_2023, crimes_2024)
crimes <- crimes %>% select(-X)
```

With the format of the `Date` column we needed to change it from a string to a Date datatype to be able to do time and date analysis.

``` r
crimes$Date <- mdy_hms(crimes$Date)
```

We then converted the data type of the `Arrest` and `Domestic` columns from strings to Boolean values.

``` r
crimes$Arrest <- as.logical(crimes$Arrest)
crimes$Domestic <- as.logical(crimes$Domestic)
```

We also wanted to get an idea of potential future issues based off of NA values, so we created a table showing the count of NA values for each column.

``` r
crimes %>%
  summarise(across(everything(), ~ sum(is.na(.))))
```

This showed us a few incidents do not have a recorded `Ward`, and although a small percentage of the over 1 million incidents, there are thousands of locations unaccounted for. However with the dataset size we are confident this will not make a major impact on results.

### Variables
* **Date**: The date and time of the incident
* **Block**: Partially redacted address which keeps the correct block
* **Primary.Type**: The primary crime description
* **Description**: The secondary crime description
* **Location.Description**: Description of the incident location
* **Arrest**: Whether an arrest was made
* **Domestic**: Whether the incident was domestic
* **District**: Police district where the incident occurred
* **Ward**: City Council district where the incident occurred
* **Year**: The year of the incident
* **Latitude**: Latitude of the incident location
* **Longitude**: Longitude of the incident location
