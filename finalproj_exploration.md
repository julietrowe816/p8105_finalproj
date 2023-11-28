finalproj_exploration
================
Juliet Rowe
2023-11-26

Load libraries

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.2     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.2     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.1     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(ggplot2)
```

Load rat data

``` r
sightings = read_csv('data/Rat_Sightings_20231121.csv') |> 

  janitor::clean_names() |> 

  separate(created_date, into=c("month","e", "day","f", "year", "g", "time"), sep=c(2,3,5,6,10,11)) |> 

  select(-e,-f,-g) |> 

  mutate(date = paste(year, month, day, sep=""), 

         date = as.numeric(date)) |>  

  filter(date <= 20231031, date >= 20160101, !incident_zip <= 10000, !incident_zip >11697, !borough %in% c("Unspecified", NA)) |> 

  select(-agency, -agency_name, -complaint_type, -descriptor, -landmark, -facility_type, -park_facility_name, -vehicle_type, -taxi_company_borough, -taxi_pick_up_location, -bridge_highway_name, -road_ramp, -bridge_highway_segment, -bridge_highway_direction) |> select(unique_key, date, year, month, day, everything())
```

    ## Warning: One or more parsing issues, call `problems()` on your data frame for details,
    ## e.g.:
    ##   dat <- vroom(...)
    ##   problems(dat)

    ## Rows: 232143 Columns: 38
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (25): Created Date, Closed Date, Agency, Agency Name, Complaint Type, De...
    ## dbl  (6): Unique Key, Incident Zip, X Coordinate (State Plane), Y Coordinate...
    ## lgl  (7): Vehicle Type, Taxi Company Borough, Taxi Pick Up Location, Bridge ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Load weather data

``` r
weather_df = rnoaa::meteo_pull_monitors(
    c("USW00094728"),
    var = c("PRCP", "TMIN", "TMAX"), 
    date_min = "2016-01-01",
    date_max = "2022-12-31") |>
  mutate(
    name = recode(id, USW00094728 = "CentralPark_NY"),
    tmin = tmin / 10,
    tmax = tmax / 10) |>
  select(name, id, everything())
```

    ## using cached file: /Users/Juliet/Library/Caches/org.R-project.R/R/rnoaa/noaa_ghcnd/USW00094728.dly

    ## date created (size, mb): 2023-09-28 10:19:41.395166 (8.524)

    ## file min/max dates: 1869-01-01 / 2023-09-30

Rat sightings by latitude and longitude

Merge weather and rate data

``` r
sightings$date <- as.Date(as.character(sightings$date), format = "%Y%m%d")

rat_weather =
  right_join(sightings, weather_df, by="date")
```
