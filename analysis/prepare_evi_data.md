``` r
library("dplyr")
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library("lubridate")
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

``` r
library("ggplot2")
```

Read and prepare evi data
-------------------------

We created two datasets:

-   annual evi by pixel
-   seasonal evi by pixel

Each dataframe has the following fields:

-   `iv_malla_modi_id`: the identifier of the modis cell
-   `year`
-   `evi_mean`: the value of the evi (cumulative value for each season)
-   `season`: the season of cumulative evi:

-   `0` annual value
-   `1` spring value
-   `2` summer value
-   `3` autumn value
-   `4` winter value
-   `seasonF`: the season coded as factor
-   `lng`: longitude coordinates
-   `lat`: latitute coordinates
-   `poblacion`: numeric code of the *Q. pyrenaica* population

``` r
# Read and prepare data
rawdata <- read.csv(file=paste(di, "/data_raw/evi/raw_iv.csv", sep= ""), header = TRUE, sep = ',')

# Create variables of year and month 
# Apply scale factor https://lpdaac.usgs.gov/dataset_discovery/modis/modis_products_table/mod13q1 
rawdata <- rawdata %>% 
  mutate(myevi = evi * 0.0001,
         year = lubridate::year(fecha),
         month = lubridate::month(fecha))

# Create annual evi by pixel 
eviyear <- rawdata %>% 
  group_by(iv_malla_modi_id, year) %>%
  summarise(evi_mean = sum(myevi[myevi >=0])) %>%
  mutate(season=0)


# Create seasonal evi 

# Julian date 
# >81 <=173 --> spring (1)
# >173 <=265 --> summer (2)
# > 265 <=356 --> autum (3)
# > 356 and < 81 --> winter (4)
# Get julian day of limits of the season
sp <-lubridate::yday(as.Date("2000-03-21"))
su <- lubridate::yday(as.Date("2000-06-21"))
au <-  lubridate::yday(as.Date("2000-09-21"))
wi <- lubridate::yday(as.Date("2000-12-21"))
season_julian <- c(sp,su,au,wi)

eviseason <- rawdata %>%
  mutate(jday=lubridate::yday(fecha)) %>%
  select(iv_malla_modi_id, year, myevi, jday) %>% 
  mutate(season = ifelse(jday > 81 & jday <= 173, 1,
                        ifelse(jday > 173 & jday <= 265, 2, 
                               ifelse(jday > 265 & jday <= 356, 3, 4)))) %>%
  group_by(iv_malla_modi_id, year, season) %>%
  summarise(evi_mean= sum(myevi[myevi >=0]))


evidf <- rbind(eviyear, eviseason)

evidf <- evidf %>% 
  mutate(seasonF = ifelse (season == 0, 'annual',
                           ifelse(season == 1, 'spring',
                                  ifelse(season == 2, 'summer',
                                         ifelse(season == 3, 'autumn', 'winter')))))



# Add coordinates and pob 
evi_aux <- rawdata %>% select(iv_malla_modi_id, lng, lat, poblacion) %>%
  group_by(iv_malla_modi_id) %>% unique()


# Join dataframes 
evi <- evidf %>% inner_join(evi_aux, by="iv_malla_modi_id") 

# Export evi dataframe
write.csv(evi, file=paste(di, "/data/evi_attributes_all.csv", sep=""), row.names = FALSE)
```