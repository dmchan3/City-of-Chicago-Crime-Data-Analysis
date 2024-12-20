City of Chicago Crime Data Analysis Project
================
11-03-24

## Research Topic

This project will analyze City of Chicago crime data to explore patterns
and insights over the past 5 years. Using this large dataset (200-300k
records per year), our aim is to identify trends and examine crime
intensity based on locations, times, and types of offenses.
Specifically, we will explore questions such as which blocks or streets
are the most dangerous, seasonal crime patterns, and how these trends
have evolved.

## Team Members

- David Chan
- Croix Westbrock
- Srishti Nandal
- Matthew Ritland

## Data

### Description of the Dataset

Our primary dataset comes from the City of Chicago government open data
portal, containing records of reported crimes. The link to the full
dataset (2001-Present) is
<https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2/about_data>
. For this project, we are using a subset of this dataset to focus on
the last 5 years. CSV files for specific years, as we will be using, can
be found on the site linked above by searching “Crimes - *Year*”.

### Questions to be Addressed

- What are the most committed crimes in Chicago? Which crimes lead to
  the most arrests? How many crimes are domestic versus non-domestic?
- Are there any seasonal crime patterns? Does quantity or types of
  crimes vary noticeably with changing of months and seasons? Time of
  day?
- How was Chicago crime affected by COVID-19? Is there a noticeable
  difference in the quantity and/or types of crimes committed during
  COVID-19 times versus recent “normal” times?
- What areas of Chicago produce the most crimes? Block? District?
- O Block is widely known as one of the most dangerous blocks in Chicago
  and the United States as a whole. How do O Block crime rates compare
  to the rest of Chicago?
- What are the top 10 most dangerous location types? Is there any change
  by year?
- What areas of Chicago hold the most arrests? Does this differ from the
  whole dataset?
- What is the most common street crime? Which block has the most street
  crime?

### Initial Data Cleaning Steps

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

While trying to load the datasets for each year, we ran into a problem
where each of the files were too large for Github’s 25mb file size
limit. To work around this, we worked in a separate R script file to
first eliminate the columns that were repetitive or not useful to our
project. This narrowed our data to 12 columns from the previous 22. We
then split each csv file into 2 different csv files, with the first
including the first 4 columns, and the second containing the last 8.
This split made each file similar in size, and more importantly, under
the Github file size threshold. The code for this process for the 2024
dataset is given below as an example (with everything commented out as
it will not run here)

``` r
# data2024 <- read.csv("ProjectFiles/Crimes - 2024.csv")
# c1_2024 <- data2024 %>% select(Date, Block, Primary.Type, Description)
# c2_2024 <- data2024 %>% select(Location.Description, Arrest, Domestic, District, Ward, Year, Latitude, Longitude)
# write.csv(c1_2024, file = "./ProjectFiles/2024_1.csv")
# write.csv(c2_2024, file = "./ProjectFiles/2024_2.csv")
```

We then merged the 2 subfiles back into one single year file to be used
here.

``` r
crimes_2020 <- merge(read.csv("2020_1.csv"), read.csv("2020_2.csv"))
crimes_2021 <- merge(read.csv("2021_1.csv"), read.csv("2021_2.csv"))
crimes_2022 <- merge(read.csv("2022_1.csv"), read.csv("2022_2.csv"))
crimes_2023 <- merge(read.csv("2023_1.csv"), read.csv("2023_2.csv"))
crimes_2024 <- merge(read.csv("2024_1.csv"), read.csv("2024_2.csv"))
```

We then needed to combine all years into one main dataframe for analysis
and remove the `X` column, which acted as the row number for records in
each individual year dataset.

``` r
crimes <- rbind(crimes_2020, crimes_2021, crimes_2022, crimes_2023, crimes_2024)
crimes <- crimes %>% select(-X)
```

With the format of the `Date` column we needed to change it from a
string to a Date datatype to be able to do time and date analysis.

``` r
crimes$Date <- mdy_hms(crimes$Date)
```

We then converted the data type of the `Arrest` and `Domestic` columns
from strings to Boolean values.

``` r
crimes$Arrest <- as.logical(crimes$Arrest)
crimes$Domestic <- as.logical(crimes$Domestic)
```

We also wanted to get an idea of potential future issues based off of NA
values, so we created a table showing the count of NA values for each
column.

``` r
crimes %>%
  summarise(across(everything(), ~ sum(is.na(.))))
```

    ##   Date Block Primary.Type Description Location.Description Arrest Domestic
    ## 1    0     0            0           0                    0      0        0
    ##   District Ward Year Latitude Longitude
    ## 1        0   33    0    16486     16486

This shows us a few incidents do not have a recorded `Ward`, and
although a small percentage of the over 1 million incidents, there are
thousands of locations unaccounted for. However with the dataset size we
are confident this will not make a major impact on results.

Regarding the table above, `Location.Description` is listed to have no
NA values, however has instances of ““. We would like to convert all
these to NA values, as that is ultimately what they are.

``` r
crimes$Location.Description <-  replace(crimes$Location.Description, crimes$Location.Description == "", NA)
```

``` r
head(crimes)
```

    ##                  Date                Block               Primary.Type
    ## 1 2020-10-30 16:30:00      011XX E 82ND ST    CRIMINAL SEXUAL ASSAULT
    ## 2 2020-10-01 00:01:00      031XX W 53RD PL OFFENSE INVOLVING CHILDREN
    ## 3 2020-09-04 00:00:00     0000X W 112TH PL OFFENSE INVOLVING CHILDREN
    ## 4 2020-08-06 00:00:00     081XX S KNOX AVE                    BATTERY
    ## 5 2020-07-18 22:00:00  067XX N WESTERN AVE                    BATTERY
    ## 6 2020-03-19 05:50:00 058XX W WAVELAND AVE          WEAPONS VIOLATION
    ##                                           Description Location.Description
    ## 1                                           PREDATORY            RESIDENCE
    ## 2            SEXUAL ASSAULT OF CHILD BY FAMILY MEMBER            RESIDENCE
    ## 3 AGGRAVATED SEXUAL ASSAULT OF CHILD BY FAMILY MEMBER            RESIDENCE
    ## 4                      AGGRAVATED OF A SENIOR CITIZEN            RESIDENCE
    ## 5                             DOMESTIC BATTERY SIMPLE        PARK PROPERTY
    ## 6                          RECKLESS FIREARM DISCHARGE            RESIDENCE
    ##   Arrest Domestic District Ward Year Latitude Longitude
    ## 1   TRUE     TRUE        4    8 2020 41.74588 -87.59717
    ## 2  FALSE     TRUE        9   14 2020       NA        NA
    ## 3   TRUE     TRUE        5    9 2020       NA        NA
    ## 4  FALSE     TRUE        8   18 2020       NA        NA
    ## 5   TRUE     TRUE       24   40 2020 42.00420 -87.69005
    ## 6  FALSE    FALSE       16   38 2020 41.94764 -87.77238

Here is the head of our dataframe that we will be using for analysis.

### Variables

- **Date**: The date and time of the incident
- **Block**: Partially redacted address which keeps the correct block
- **Primary.Type**: The primary crime description
- **Description**: The secondary crime description
- **Location.Description**: Description of the incident location
- **Arrest**: Whether an arrest was made
- **Domestic**: Whether the incident was domestic
- **District**: Police district where the incident occurred
- **Ward**: City Council district where the incident occurred
- **Year**: The year of the incident
- **Latitude**: Latitude of the incident location
- **Longitude**: Longitude of the incident location

#### What are the most committed crimes in Chicago? Which crimes lead to the most arrests? How many crimes are domestic versus non-domestic?

-Firstly we identify the most frequently reported crimes in Chicago.

``` r
# Count the frequency of each crime type
crime_counts <- crimes %>%
  group_by(Primary.Type) %>%
  summarise(Count = n()) %>%
  arrange(desc(Count))
# Display top 10 most common crimes
crime_counts_top10 <- head(crime_counts, 10)
print(crime_counts_top10)
```

    ## # A tibble: 10 × 2
    ##    Primary.Type         Count
    ##    <chr>                <int>
    ##  1 THEFT               243322
    ##  2 BATTERY             204804
    ##  3 CRIMINAL DAMAGE     130570
    ##  4 ASSAULT             101387
    ##  5 MOTOR VEHICLE THEFT  89264
    ##  6 DECEPTIVE PRACTICE   82417
    ##  7 OTHER OFFENSE        70587
    ##  8 ROBBERY              43234
    ##  9 WEAPONS VIOLATION    41576
    ## 10 BURGLARY             37067

``` r
# Plot the top 10 crimes
ggplot(crime_counts_top10, aes(x = reorder(Primary.Type, -Count), y = Count)) +
  geom_bar(stat = "identity", fill = "steelblue") +
  labs(title = "Top 10 Most Committed Crimes in Chicago", x = "Crime Type", y = "Count") +
  theme_minimal() +
  coord_flip()
```

![](README_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

Theft is the most common crime, followed by battery and criminal damage.

-Then identify which crimes have the highest arrest rates.

``` r
# Calculate arrest rates by crime type
ArrestRates <- crimes %>%
  group_by(Primary.Type) %>%
  summarise(Total = n(), Arrests = sum(Arrest, na.rm = TRUE)) %>%
  mutate(ArrestRate = Arrests / Total * 100) %>%
  arrange(desc(ArrestRate))

# Display top 10 crimes with the highest arrest rates
top10 <- head(ArrestRates, 10)
print(top10)
```

    ## # A tibble: 10 × 4
    ##    Primary.Type                      Total Arrests ArrestRate
    ##    <chr>                             <int>   <int>      <dbl>
    ##  1 LIQUOR LAW VIOLATION                877     860       98.1
    ##  2 GAMBLING                             80      78       97.5
    ##  3 NARCOTICS                         27790   27060       97.4
    ##  4 PROSTITUTION                       1099    1068       97.2
    ##  5 CONCEALED CARRY LICENSE VIOLATION   869     838       96.4
    ##  6 PUBLIC INDECENCY                     33      30       90.9
    ##  7 INTERFERENCE WITH PUBLIC OFFICER   2502    2209       88.3
    ##  8 OBSCENITY                           240     152       63.3
    ##  9 WEAPONS VIOLATION                 41576   25372       61.0
    ## 10 OTHER NARCOTIC VIOLATION             22      11       50

``` r
# Plot crimes with the highest arrest rates
ggplot(top10, aes(x = reorder(Primary.Type, -ArrestRate), y = ArrestRate)) +
  geom_bar(stat = "identity", fill = "skyblue") +
  labs(title = "Crimes with the Highest Arrest Rates", x = "Crime Type", y = "Arrest Rate (%)") +
  theme_minimal() +
  coord_flip()
```

![](README_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

Narcotics-related offenses lead in arrest rates, reflecting targeted
policing. Violent crimes like homicide and robbery also show high arrest
rates due to their severity and societal focus.

Now Comparing the frequency of domestic versus non-domestic crimes.

``` r
library(dplyr)
# Count domestic and non-domestic crimes
DomesticCounts <- crimes %>%
  group_by(Domestic) %>%
  summarise(Count = n())

print(DomesticCounts)
```

    ## # A tibble: 2 × 2
    ##   Domestic  Count
    ##   <lgl>     <int>
    ## 1 FALSE    909081
    ## 2 TRUE     224539

``` r
# Pie chart for domestic vs. non-domestic crimes
ggplot(DomesticCounts, aes(x = "", y = Count, fill = Domestic)) +
  geom_bar(stat = "identity", width = 1) +
  coord_polar("y") +
  labs(title = "Proportion of Domestic vs. Non-Domestic Crimes", fill = "Domestic") +
  theme_void()
```

![](README_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

``` r
# Load required libraries
library(tidyverse)  # For data manipulation and visualization
library(lubridate)  # For working with dates and times
library(ggplot2)    # For plotting
library(scales)     # For numeric formatting in plots
```

    ## 
    ## Attaching package: 'scales'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     discard

    ## The following object is masked from 'package:readr':
    ## 
    ##     col_factor

``` r
# List of years to process
years <- 2020:2024

# Function to load and merge `_1` and `_2` files for a given year
load_and_merge <- function(year) {
  file1 <- paste0(year, "_1.csv")
  file2 <- paste0(year, "_2.csv")
  
  # Read the two parts of the data
  data1 <- read.csv(file1)
  data2 <- read.csv(file2)
  
  # Align column names dynamically
  missing_in_data2 <- setdiff(colnames(data1), colnames(data2))
  missing_in_data1 <- setdiff(colnames(data2), colnames(data1))
  
  # Add missing columns
  if (length(missing_in_data2) > 0) {
    for (col in missing_in_data2) {
      data2[[col]] <- NA
    }
  }
  if (length(missing_in_data1) > 0) {
    for (col in missing_in_data1) {
      data1[[col]] <- NA
    }
  }
  
  # Reorder columns in data2 to match data1
  data2 <- data2[colnames(data1)]
  
  # Merge the two datasets
  merged_data <- bind_rows(data1, data2)
  
  return(merged_data)
}

# Load and merge data for all years
crime_data <- map_dfr(years, load_and_merge)

# Data Cleaning
crime_data <- crime_data %>%
  select(Date, Primary.Type, Location.Description, Arrest, Domestic, Latitude, Longitude) %>%
  mutate(Date = mdy_hms(Date)) %>%  # Convert Date column to datetime
  filter(!is.na(Date))  # Remove rows with invalid dates

# Add time-related components
crime_data <- crime_data %>%
  mutate(
    Month = month(Date, label = TRUE, abbr = TRUE),
    Season = case_when(
      Month %in% c("Dec", "Jan", "Feb") ~ "Winter",
      Month %in% c("Mar", "Apr", "May") ~ "Spring",
      Month %in% c("Jun", "Jul", "Aug") ~ "Summer",
      Month %in% c("Sep", "Oct", "Nov") ~ "Fall"
    ),
    Hour = hour(Date)
  )

# Analyze monthly crime patterns
monthly_crime <- crime_data %>%
  group_by(Month) %>%
  summarize(Crime_Count = n())

# Analyze seasonal crime patterns
seasonal_crime <- crime_data %>%
  group_by(Season) %>%
  summarize(Crime_Count = n())

# Analyze time-of-day crime patterns
hourly_crime <- crime_data %>%
  group_by(Hour) %>%
  summarize(Crime_Count = n())

# Analyze types of crimes by season
crime_types_by_season <- crime_data %>%
  group_by(Season, Primary.Type) %>%
  summarize(Crime_Count = n()) %>%
  arrange(Season, desc(Crime_Count))
```

    ## `summarise()` has grouped output by 'Season'. You can override using the
    ## `.groups` argument.

``` r
# Visualizations

# Monthly crime trends
ggplot(monthly_crime, aes(x = Month, y = Crime_Count, group = 1)) +
  geom_line() +
  geom_point() +
  labs(
    title = "Monthly Crime Trends",
    x = "Month",
    y = "Crime Count"
  )
```

![](README_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

``` r
# Seasonal crime patterns
ggplot(seasonal_crime, aes(x = Season, y = Crime_Count, fill = Season)) +
  geom_bar(stat = "identity") +
  scale_y_continuous(labels = scales::comma) +
  labs(
    title = "Seasonal Crime Distribution",
    x = "Season",
    y = "Crime Count"
  ) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

![](README_files/figure-gfm/unnamed-chunk-13-2.png)<!-- -->

``` r
# Hourly crime patterns
ggplot(hourly_crime, aes(x = Hour, y = Crime_Count)) +
  geom_line() +
  labs(
    title = "Hourly Crime Trends",
    x = "Hour of Day",
    y = "Crime Count"
  )
```

![](README_files/figure-gfm/unnamed-chunk-13-3.png)<!-- -->

``` r
# Top crime types by season
ggplot(crime_types_by_season, aes(x = Primary.Type, y = Crime_Count, fill = Season)) +
  geom_bar(stat = "identity", position = "dodge") +
  scale_y_continuous(labels = scales::comma) +
  labs(
    title = "Top Crime Types by Season",
    x = "Crime Type",
    y = "Crime Count"
  ) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

![](README_files/figure-gfm/unnamed-chunk-13-4.png)<!-- -->

**Data Loading**: I read crime data files for each year, handling files
split into two parts while ensuring specific files, like 2020_1.csv, are
excluded as required.

**Data Cleaning and Preprocessing**: I addressed missing values, aligned
column structures for consistency, and extracted time-related features
such as months, seasons, and hours from the date information.

**Analysis**: I aggregated crime data to calculate counts by month,
season, and hour of the day. I identified and ranked the most common
crime types for each season.

**Visualizations**: I created clear and informative visualizations,
including: Trends in crime counts by month, season, and hour.
Comparisons of crime types by season using grouped bar charts.

#### How was Chicago crime affected by COVID-19? Is there a noticeable difference in the quantity and/or types of crimes committed during COVID-19 times versus recent “normal” times?

- Analyzing crime trends during COVID-19 (March 2020–June 2021).

``` r
# Define periods
crimes <- crimes %>%
  mutate(COVID_Period = case_when(
    Date >= ymd("2020-03-01") & Date <= ymd("2021-06-30") ~ "COVID-19",
    Date > ymd("2021-06-30") ~ "Post-COVID",
    Date < ymd("2020-03-01") ~ "Pre-COVID"
  ))

# Compare total crimes across periods
crime_period_counts <- crimes %>%
  group_by(COVID_Period) %>%
  summarise(Count = n(), .groups = "drop")

print(crime_period_counts)
```

    ## # A tibble: 3 × 2
    ##   COVID_Period  Count
    ##   <chr>         <int>
    ## 1 COVID-19     271665
    ## 2 Post-COVID   823713
    ## 3 Pre-COVID     38242

``` r
# Bar plot for crime counts across periods
ggplot(crime_period_counts, aes(x = COVID_Period, y = Count, fill = COVID_Period)) +
  geom_bar(stat = "identity") +
  labs(title = "Crime Counts Across Periods", x = "Period", y = "Crime Count") +
  theme_minimal()
```

![](README_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

``` r
# Compare crime types during COVID-19
crime_type_covid <- crimes %>%
  filter(COVID_Period %in% c("COVID-19", "Pre-COVID")) %>%
  group_by(COVID_Period, Primary.Type) %>%
  summarise(Count = n(), .groups = "drop") %>%
  arrange(COVID_Period, desc(Count))

print(head(crime_type_covid, 20))
```

    ## # A tibble: 20 × 3
    ##    COVID_Period Primary.Type                     Count
    ##    <chr>        <chr>                            <int>
    ##  1 COVID-19     BATTERY                          53328
    ##  2 COVID-19     THEFT                            49559
    ##  3 COVID-19     CRIMINAL DAMAGE                  32904
    ##  4 COVID-19     DECEPTIVE PRACTICE               25522
    ##  5 COVID-19     ASSAULT                          24674
    ##  6 COVID-19     OTHER OFFENSE                    17000
    ##  7 COVID-19     MOTOR VEHICLE THEFT              13120
    ##  8 COVID-19     WEAPONS VIOLATION                11947
    ##  9 COVID-19     BURGLARY                         10011
    ## 10 COVID-19     ROBBERY                           9724
    ## 11 COVID-19     NARCOTICS                         8496
    ## 12 COVID-19     CRIMINAL TRESPASS                 4620
    ## 13 COVID-19     OFFENSE INVOLVING CHILDREN        2548
    ## 14 COVID-19     CRIMINAL SEXUAL ASSAULT           1695
    ## 15 COVID-19     PUBLIC PEACE VIOLATION            1349
    ## 16 COVID-19     SEX OFFENSE                       1277
    ## 17 COVID-19     HOMICIDE                          1066
    ## 18 COVID-19     ARSON                              783
    ## 19 COVID-19     INTERFERENCE WITH PUBLIC OFFICER   578
    ## 20 COVID-19     STALKING                           339

``` r
# Prepare data for the final bar graph
crime_type_diff <- crime_type_covid %>%
  pivot_wider(names_from = COVID_Period, values_from = Count, values_fill = 0) %>%
  mutate(Difference = `COVID-19` - `Pre-COVID`) %>%
  arrange(desc(Difference)) %>%
  slice(1:10)  # Select top 10 crime types with the largest difference

print(crime_type_diff)
```

    ## # A tibble: 10 × 4
    ##    Primary.Type        `COVID-19` `Pre-COVID` Difference
    ##    <chr>                    <int>       <int>      <int>
    ##  1 BATTERY                  53328        6995      46333
    ##  2 THEFT                    49559        8785      40774
    ##  3 CRIMINAL DAMAGE          32904        3534      29370
    ##  4 DECEPTIVE PRACTICE       25522        3065      22457
    ##  5 ASSAULT                  24674        2906      21768
    ##  6 OTHER OFFENSE            17000        2670      14330
    ##  7 MOTOR VEHICLE THEFT      13120        1322      11798
    ##  8 WEAPONS VIOLATION        11947         951      10996
    ##  9 BURGLARY                 10011        1359       8652
    ## 10 ROBBERY                   9724        1363       8361

``` r
# Bar plot showing the differences
ggplot(crime_type_diff, aes(x = reorder(Primary.Type, Difference), y = Difference, fill = Difference > 0)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  labs(title = "Top 10 Crime Type Differences (COVID-19 vs Pre-COVID)", 
       x = "Crime Type", y = "Difference (COVID-19 - Pre-COVID)") +
  scale_fill_manual(values = c("TRUE" = "forestgreen", "FALSE" = "maroon"),
                    labels = c("Increase", "Decrease")) +
  theme_minimal()
```

![](README_files/figure-gfm/unnamed-chunk-14-2.png)<!-- -->

``` r
# Prepare data for the final bar graph
crime_type_diff <- crime_type_covid %>%
  pivot_wider(names_from = COVID_Period, values_from = Count, values_fill = 0) %>%
  arrange(desc(`COVID-19`)) %>%
  slice(1:10)  # Select top 10 crime types based on COVID-19 counts

# Reshape data for side-by-side comparison
crime_type_diff_long <- crime_type_diff %>%
  pivot_longer(cols = c(`COVID-19`, `Pre-COVID`), names_to = "Time_Period", values_to = "Count")

# Create side-by-side bar plot
library(ggplot2)

ggplot(crime_type_diff_long, aes(x = reorder(Primary.Type, Count), y = Count, fill = Time_Period)) +
  geom_bar(stat = "identity", position = "dodge") +
  coord_flip() +
  labs(title = "Crime Counts: COVID-19 vs. Pre-COVID",
       x = "Crime Type", y = "Crime Count") +
  scale_fill_manual(values = c("COVID-19" = "forestgreen", "Pre-COVID" = "maroon")) +
  theme_minimal() +
  theme(legend.title = element_blank())
```

![](README_files/figure-gfm/unnamed-chunk-14-3.png)<!-- -->

#### What areas of Chicago produce the most crimes? Block? District?

``` r
library(dplyr)
library(ggplot2)


# finding which blocks have the highest frequency of crime
block_crime_summary <- crimes %>%
  group_by(Block) %>%
  summarize(Crime_Count = n()) %>%
  arrange(desc(Crime_Count))

# selecting top 10 blocks with highest crimes committed
top_blocks <- block_crime_summary %>%
  top_n(10, Crime_Count)

# creating graph showing top 10 blocks with most crimes
ggplot(top_blocks, aes(x = reorder(Block, -Crime_Count), y = Crime_Count)) +
  geom_bar(stat = "identity", fill = "blue") +
  coord_flip() +
  labs(title = "Top 10 Blocks with Most Crimes", x = "Block", y = "Crime Count") +
  theme_minimal()
```

![](README_files/figure-gfm/unnamed-chunk-15-1.png)<!-- --> By creating
this graph, it is much easier to visually conclude which blocks have the
highest amount of crime.

``` r
# finding which districts have the highest frequency of crime
district_crime_summary <- crimes %>%
  group_by(District) %>%
  summarize(Crime_Count = n()) %>%
  arrange(desc(Crime_Count))


# creating graph showing most to least crimes by disctrict
ggplot(district_crime_summary, aes(x = reorder(District, -Crime_Count), y = Crime_Count)) +
  geom_bar(stat = "identity", fill = "red") +
  labs(title = "Crimes by District", x = "District", y = "Crime Count") +
  theme_minimal()
```

![](README_files/figure-gfm/unnamed-chunk-16-1.png)<!-- --> From this
graph it is easy to see which districts produce the most crime over the
last few years.

#### O Block is widely known as one of the most dangerous blocks in Chicago and the United States as a whole. How do O Block crime rates compare to the rest of Chicago?

Firstly, we have to define what crimes happened in O Block. To do this,
we looked at the coordinates at two corners of O Block in Google Maps to
determine the longitude and latitude range of O Block.

``` r
# Filter O Block crimes
oblock <- crimes %>% 
  filter(Latitude < 41.78037, Latitude > 41.774810, 
         Longitude > -87.617464, Longitude < -87.614991)

# Make dataset for rest of Chicago (whole dataset - O Block dataset)
non_oblock <- crimes %>% 
  anti_join(oblock)
```

    ## Joining with `by = join_by(Date, Block, Primary.Type, Description,
    ## Location.Description, Arrest, Domestic, District, Ward, Year, Latitude,
    ## Longitude, COVID_Period)`

We will first begin with an overall analysis of O Block crimes.

``` r
#O Block is stated to have a capacity of 2,000 people
oblock_population = 2000

# Create a table of the crime counts, percentages of total crimes, and rate per 100 people for the O Block data
obrates <- oblock %>% 
  group_by(Primary.Type) %>% 
  summarise(
    n = n(),
    percent = n() / nrow(oblock) * 100,
    rate_per_100 = n() / oblock_population * 100
  ) %>% 
  arrange(desc(n))

obrates
```

    ## # A tibble: 21 × 4
    ##    Primary.Type            n percent rate_per_100
    ##    <chr>               <int>   <dbl>        <dbl>
    ##  1 BATTERY               913   34.2         45.6 
    ##  2 ASSAULT               323   12.1         16.2 
    ##  3 CRIMINAL DAMAGE       316   11.8         15.8 
    ##  4 THEFT                 255    9.56        12.8 
    ##  5 WEAPONS VIOLATION     149    5.59         7.45
    ##  6 OTHER OFFENSE         136    5.10         6.8 
    ##  7 ROBBERY               107    4.01         5.35
    ##  8 CRIMINAL TRESPASS      89    3.34         4.45
    ##  9 MOTOR VEHICLE THEFT    84    3.15         4.2 
    ## 10 BURGLARY               74    2.77         3.7 
    ## # ℹ 11 more rows

``` r
# Create a bar chart visualization for the rate per 100 people column of the above table
obrates %>% 
  ggplot(
    aes(
      x = reorder(Primary.Type, rate_per_100), 
      y = rate_per_100
    )
  ) + 
  geom_col(fill = "red") +
  coord_flip() +
  ggtitle("O Block Crime Rates") +
  xlab("Crime") +
  ylab("Rate Per 100 People") + theme_bw()
```

![](README_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->

Now we will look at the same analysis of non O Block crimes.

``` r
# The city of Chicago is listed to have a population of 2.664 million, however that includes O Block
non_oblock_population = 2664000 - oblock_population

# Create a table of the crime counts, percentages of total crimes, and rate per 100 people for the non-O Block data
nonrates <- non_oblock %>% 
  group_by(Primary.Type) %>% 
  summarise(
    n = n(),
    percent = n() / nrow(non_oblock) * 100,
    rate_per_100 = n() / non_oblock_population * 100
  ) %>% 
  arrange(desc(n))

nonrates
```

    ## # A tibble: 33 × 4
    ##    Primary.Type             n percent rate_per_100
    ##    <chr>                <int>   <dbl>        <dbl>
    ##  1 THEFT               243067   21.5          9.13
    ##  2 BATTERY             203891   18.0          7.66
    ##  3 CRIMINAL DAMAGE     130254   11.5          4.89
    ##  4 ASSAULT             101064    8.94         3.80
    ##  5 MOTOR VEHICLE THEFT  89180    7.89         3.35
    ##  6 DECEPTIVE PRACTICE   82356    7.28         3.09
    ##  7 OTHER OFFENSE        70451    6.23         2.65
    ##  8 ROBBERY              43127    3.81         1.62
    ##  9 WEAPONS VIOLATION    41427    3.66         1.56
    ## 10 BURGLARY             36993    3.27         1.39
    ## # ℹ 23 more rows

``` r
# Create a bar chart visualization for the  crime rates per 100 people of the above table
nonrates %>% 
  filter(
    rate_per_100 > .01
  ) %>% 
  ggplot(
    aes(
      x = reorder(Primary.Type, rate_per_100), 
      y = rate_per_100
    )
  ) + 
  geom_col(fill = "red") +
  coord_flip() +
  ggtitle("Rest of Chicago Crime Rates") +
  xlab("Crime") +
  ylab("Rate Per 100 People") + theme_bw()
```

![](README_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

We see in the tables above that O Block holds a greater percentage of
its crimes in battery and assault, more violent crimes, whereas we see a
higher percentage of theft related crimes. The largest difference we see
in these tables come in the `rate_per_100` column. This column shows how
many instances of each crime occur per 100 people in the population.
Here we see overall rates significantly higher in O Block than the rest
of Chicago, especially in violent crimes. In O Block, we see that for
every 100 people, there are 45.65 batteries, 16.15 assaults, and 0.70
homicides. Additionally, we see extremely high rates in criminal damage
(15.80), and theft (12.75). Comparing the violent crimes to the rest of
Chicago, in the rest of Chicago we see per 100 people only approximately
7.66 batteries, 3.80 assaults, and 0.13 homicides. Even theft, which has
the highest rate in non O Block Chicago, still has a lower rate than O
Block, with 9.13 thefts per 100 people, and criminal damage in third is
less than a third of O Block’s rate, at 4.89.

Overall, we see significantly higher crime rates in O Block than the
rest of Chicago. More specifically, we see even higher crime rate
differences in violent crimes.

#### What are the most dangerous location types? Is there any change by year?

We will begin with looking at overall most dangerous location types by
total count across all 5 years.

``` r
# Gets and visualizes the top 10 location types with the most total crime across the whole dataset.
crimes %>% 
  drop_na(Location.Description) %>% 
  group_by(Location.Description) %>% 
  summarise(
    count = n()
  ) %>% 
  arrange(desc(count)) %>% 
  top_n(10, count) %>% 
  ggplot(
    aes(
      x = count / 1000,
      y = reorder(Location.Description, count),
    )
  ) + geom_col(fill = "skyblue") +
  ggtitle("Most Dangerous Overall Location Types") +
  xlab("Count (Thousands)") +
  ylab("Location Type") + theme_bw()
```

![](README_files/figure-gfm/unnamed-chunk-22-1.png)<!-- -->

This graph shows that the most crimes occur on/in streets, followed by
apartments and overall residences. We see these three be significantly
greater than all the other location types.

We will now look at a yearly breakdown of this question.

``` r
# Table of top 5 location descriptions with the most crimes committed by year
t5_by_year <- crimes %>% 
  group_by(Year, Location.Description) %>% 
  summarise(
    count = n()
  ) %>% 
  arrange(desc(count)) %>% 
  group_by(Year) %>% 
  slice(1:5)
```

    ## `summarise()` has grouped output by 'Year'. You can override using the
    ## `.groups` argument.

``` r
# Creating a line + scatterplot plot visualizing the above table
t5_by_year %>% 
  ggplot(
    aes(
      x = Year,
      y = count,
      colour = Location.Description
    )
  ) + geom_line() + geom_point() + theme_bw() +
  ggtitle("Location Types With Highest Crime Counts By Year") +
  xlab("Year") +
  ylab("Count")
```

![](README_files/figure-gfm/unnamed-chunk-23-1.png)<!-- -->

In the graph above, we see that the same three categories; street,
apartment, and residence, are year after year the top 3 without much
challenge. Sidewalks come fourth year in and year out, sometimes
narrowly. A surprise we see in this graph is the presence of small
retail stores having the fifth highest crime counts in 2020. This comes
as a shock as 2020 was COVID times, and on March 21, the state of
Illinois announced a statewide Stay At Home Order. With this coming into
effect after less than 3 months of the year, it comes at a great
surprise that small retail stores would still come fifth in location
based crime counts.

#### What areas of Chicago hold the most arrests? Does this differ from the whole dataset?

``` r
# Load required libraries
library(tidyverse)  # For data manipulation and visualization
library(scales)     # For numeric formatting in plots

# Define years and file parts to process
years <- 2020:2024
file_parts <- c("_1.csv", "_2.csv")

# Function to load and clean data
load_and_clean <- function(year, part) {
  file <- paste0(year, part)  # Construct file name
  
  if (file == "2020_1.csv") {  # Exclude 2020_1.csv
    return(NULL)
  }
  
  # Load data and handle errors
  data <- tryCatch({
    read.csv(file)
  }, error = function(e) {
    warning(paste("Could not load file:", file))
    return(NULL)
  })
  
  # Add placeholder columns if missing
  if (!"Arrest" %in% colnames(data)) data$Arrest <- NA
  if (!"District" %in% colnames(data)) data$District <- NA
  
  # Convert Arrest column to logical
  data <- data %>%
    mutate(
      Arrest = case_when(
        Arrest %in% c("TRUE", "true", "1") ~ TRUE,
        Arrest %in% c("FALSE", "false", "0") ~ FALSE,
        TRUE ~ NA
      )
    ) %>%
    filter(!is.na(District) & !is.na(Arrest))  # Keep valid rows
  
  return(data)
}

# Combine data from all files
crime_data <- map_dfr(years, function(year) {
  map_dfr(file_parts, function(part) {
    load_and_clean(year, part)
  })
})

# Count arrests by District
arrests_by_district <- crime_data %>%
  filter(Arrest == TRUE) %>%  # Filter arrests only
  group_by(District) %>%
  summarize(Total_Arrests = n()) %>%  # Count arrests
  arrange(desc(Total_Arrests))  # Sort by total arrests

# Count total crimes by District
total_crimes_by_district <- crime_data %>%
  group_by(District) %>%
  summarize(Total_Crimes = n())  # Count total crimes

# Merge arrests and total crimes, calculate arrest rate
district_comparison <- left_join(arrests_by_district, total_crimes_by_district, by = "District") %>%
  mutate(Arrest_Rate = Total_Arrests / Total_Crimes)

# Get top 10 districts with most arrests
top_arrest_districts <- district_comparison %>%
  slice_max(order_by = Total_Arrests, n = 10)

# Print top 10 districts with most arrests
print("Top 10 Districts with Most Arrests:")
```

    ## [1] "Top 10 Districts with Most Arrests:"

``` r
print(top_arrest_districts)
```

    ## # A tibble: 10 × 4
    ##    District Total_Arrests Total_Crimes Arrest_Rate
    ##       <int>         <int>        <int>       <dbl>
    ##  1       11         18313        68180       0.269
    ##  2        6          9447        70376       0.134
    ##  3        1          9166        57178       0.160
    ##  4       10          9015        48963       0.184
    ##  5        7          8313        52297       0.159
    ##  6       25          7952        58929       0.135
    ##  7        8          7595        71823       0.106
    ##  8       18          7224        55714       0.130
    ##  9        5          6782        48644       0.139
    ## 10        4          6631        64833       0.102

``` r
# Plot: Total arrests by district
ggplot(top_arrest_districts, aes(x = reorder(District, -Total_Arrests), y = Total_Arrests)) +
  geom_bar(stat = "identity", fill = "steelblue") +
  scale_y_continuous(labels = scales::comma) +  # Format Y-axis
  labs(
    title = "Top 10 Districts with Most Arrests",
    x = "District",
    y = "Total Arrests"
  ) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

![](README_files/figure-gfm/unnamed-chunk-24-1.png)<!-- -->

``` r
# Plot: Arrest rate by district
ggplot(top_arrest_districts, aes(x = reorder(District, -Arrest_Rate), y = Arrest_Rate)) +
  geom_bar(stat = "identity", fill = "darkgreen") +
  scale_y_continuous(labels = scales::percent) +  # Show as percentage
  labs(
    title = "Top 10 Districts by Arrest Rate",
    x = "District",
    y = "Arrest Rate"
  ) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

![](README_files/figure-gfm/unnamed-chunk-24-2.png)<!-- -->

I also did a comparison of arrests and non arrests based on the crimes
and what the rates were.

``` r
# Non_Arrest_Rate to the dataset
top_arrest_districts <- top_arrest_districts %>%
  mutate(Non_Arrest_Rate = 1 - Arrest_Rate)  # Calculate non-arrest rate

# Reshape data for rate visualization
rate_data <- top_arrest_districts %>%
  pivot_longer(
    cols = c(Arrest_Rate, Non_Arrest_Rate),  # Reshape Arrest_Rate and Non_Arrest_Rate
    names_to = "Rate_Type",                  # New column to indicate the rate type
    values_to = "Rate"                       # New column to hold the rate values
  )

# Visualization: Arrest and Non-Arrest Rates by District (Top 10)
ggplot(rate_data, aes(x = reorder(District, -Rate), y = Rate, fill = Rate_Type)) +
  geom_bar(stat = "identity", position = "dodge") +  # Side-by-side bars
  scale_y_continuous(labels = scales::percent) +     # Show Y-axis as percentages
  scale_fill_manual(
    values = c("Arrest_Rate" = "steelblue", "Non_Arrest_Rate" = "orange"),  # Assign colors
    labels = c("Arrest Rate", "Non-Arrest Rate"),                          # Legend labels
    name = "Rate Type"                                                     # Legend title
  ) +
  labs(
    title = "Arrest and Non-Arrest Rates by District (Top 10)",  # Chart title
    x = "District",                                             # X-axis label
    y = "Rate"                                                  # Y-axis label
  ) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))      # Rotate X-axis labels
```

![](README_files/figure-gfm/unnamed-chunk-25-1.png)<!-- -->

#### What is the most common street crime? Which block has the most street crime?

``` r
library(dplyr)
library(ggplot2)

# filter for street locations
street_crime_summary <- crimes %>%
  filter(Location.Description == "STREET") %>%
  group_by(Primary.Type) %>%
  summarize(Crime_Count = n()) %>%
  top_n(10, Crime_Count) %>%
  arrange(desc(Crime_Count))

# plot the most common street crime
ggplot(street_crime_summary, aes(x = reorder(Primary.Type, -Crime_Count), y = Crime_Count)) +
  geom_bar(stat = "identity", fill = "blue") +
  coord_flip() +
  labs(title = "Top 10 Most Common Crime Types for STREET Locations", x = "Crime Type", y = "Crime Count") +
  theme_minimal()
```

![](README_files/figure-gfm/unnamed-chunk-26-1.png)<!-- --> Here, it
isn’t too surprising that motor vehicle theft is the most common kind of
crime for crimes committed within a street. It is somewhat surprising
that over the last 4 years more cars were stolen than ordinary
belonging, even if theft occurs within different locations.

``` r
# find crime committed on a street grouped by block
street_crimes <- crimes %>%
  filter(Location.Description == "STREET") %>%
  group_by(Block) %>%
  summarize(Crime_Count = n()) %>%
  top_n(20, Crime_Count) %>%
  arrange(desc(Crime_Count))

# plot the top 10 streets with the highest crime
ggplot(street_crimes, aes(x = reorder(Block, -Crime_Count), y = Crime_Count)) +
  geom_bar(stat = "identity", fill = "blue") +
  coord_flip() +
  labs(title = "Top 20 Blocks with the Most Street Crime", x = "Block", y = "Crime Count") +
  theme_minimal()
```

![](README_files/figure-gfm/unnamed-chunk-27-1.png)<!-- -->

Finally, as kind of a bonus we wanted to see which block had the most
street crime. From the graph above, it is easy to tell that Martin
Luther King JR DR has the most street crime. It is interesting that from
highest crime count to lowest crime count there is a fast decline of
crime count between the streets, but then relatively quickly the crime
count becomes more or less the same for the blocks following. (For the
top 20 at least)
