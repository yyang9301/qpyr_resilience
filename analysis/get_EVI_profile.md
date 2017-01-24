Get EVI and NDVI profile for Q. pyrenaica forest
================================================

``` r
library("plyr")
library("dplyr")
library("ggplot2")
library("stringr")
library("scales")
```

Introduction
------------

-   Read raw data of EVI and NDVI attributes (see this [script](/analysis/prepare_evi_data.md))

``` r
# Read data
iv <- read.csv(file=paste(di, "/data/iv_composite.csv", sep=""), header = TRUE, sep = ',')
```

Prepare data
------------

``` r
evi_profile_dat <- iv %>% 
  group_by(composite) %>% 
  summarise(mean=mean(evi),
            sd = sd(evi),
            se = sd/sqrt(length(evi))) %>% 
  mutate(composite_dates = 
           plyr::mapvalues(composite,
                           c(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23),
                           c('01-01','01-17','02-02','02-18','03-06','03-22','04-07','04-23',
                             '05-09','05-25','06-10','06-26','07-12','07-28','08-13','08-29',
                             '09-14','09-30','10-16','11-01','11-17','12-03','12-19'))) %>%
  mutate(cd = as.Date(composite_dates, format = '%m-%d'))


ndvi_profile_dat <- iv %>% 
  group_by(composite) %>% 
  summarise(mean=mean(ndvi),
            sd = sd(ndvi),
            se = sd/sqrt(length(ndvi))) %>% 
  mutate(composite_dates = 
           plyr::mapvalues(composite,
                           c(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23),
                           c('01-01','01-17','02-02','02-18','03-06','03-22','04-07','04-23',
                             '05-09','05-25','06-10','06-26','07-12','07-28','08-13','08-29',
                             '09-14','09-30','10-16','11-01','11-17','12-03','12-19'))) %>%
  mutate(cd = as.Date(composite_dates, format = '%m-%d'))
```

Plot EVI profile
----------------

``` r
micolor <- '#455883'

evi_profile <- ggplot(evi_profile_dat, aes(cd, y=mean)) + 
  geom_errorbar(aes(ymin = mean - 10*se, ymax= mean + 10*se), width=4, colour=micolor, size=.8) + 
  #geom_errorbar(aes(ymin = mean - sd, ymax= mean + sd), width=4, colour='black') +
  geom_line(colour=micolor, size=.8) + 
  geom_point(size=3,colour=micolor) +
  geom_point(size=1.5, colour='white')+
  scale_x_date(labels = function(x) format(x, "%b"),
               breaks = date_breaks('month')) + 
  ylab('EVI') + xlab('Date') + 
  theme_bw() +
  theme(panel.grid.major=element_blank(),
        panel.grid.minor=element_blank()) +
  theme_classic()
#format(x, "%d-%b") 

print(evi_profile)
```

![](get_EVI_profile_files/figure-markdown_github/unnamed-chunk-3-1.png)

Plot NDVI profile
-----------------

``` r
ndvi_profile <- ggplot(ndvi_profile_dat, aes(cd, y=mean)) + 
  geom_errorbar(aes(ymin = mean - 10*se, ymax= mean + 10*se), width=4, colour=micolor, size=.8) + 
  #geom_errorbar(aes(ymin = mean - sd, ymax= mean + sd), width=4, colour='black') +
  geom_line(colour=micolor, size=.8) + 
  geom_point(size=3,colour=micolor) +
  geom_point(size=1.5, colour='white')+
  scale_x_date(labels = function(x) format(x, "%b"),
               breaks = date_breaks('month')) + 
  ylab('NDVI') + xlab('Date') + 
  theme_bw() +
  theme(panel.grid.major=element_blank(),
        panel.grid.minor=element_blank()) +
  theme_classic()
#format(x, "%d-%b") 

print(ndvi_profile)
```

![](get_EVI_profile_files/figure-markdown_github/unnamed-chunk-4-1.png)