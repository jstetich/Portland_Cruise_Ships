Portland Cruise Ship Visits 2015-2019
================
Curtis C. Bohlen, Casco Bay Estuary Partnership
7/16/2020

  - [Load Data](#load-data)
      - [Establish Folder Reference](#establish-folder-reference)
      - [Load Excel Data](#load-excel-data)
  - [Strategy](#strategy)
  - [Annual Totals](#annual-totals)
      - [Table](#table)
  - [Graphics for visits and
    passengers](#graphics-for-visits-and-passengers)
  - [Graphics for Ship Size](#graphics-for-ship-size)

<img
  src="https://www.cascobayestuary.org/wp-content/uploads/2014/04/logo_sm.jpg"
  style="position:absolute;top:10px;right:50px;" />

``` r
library(tidyverse)
```

    ## -- Attaching packages ------------------------------------------------------------------------------------------------------------------------------------------------------------- tidyverse 1.3.0 --

    ## v ggplot2 3.3.2     v purrr   0.3.4
    ## v tibble  3.0.1     v dplyr   1.0.0
    ## v tidyr   1.1.0     v stringr 1.4.0
    ## v readr   1.3.1     v forcats 0.5.0

    ## -- Conflicts ---------------------------------------------------------------------------------------------------------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(readxl)

library(CBEPgraphics)
load_cbep_fonts()
```

# Load Data

## Establish Folder Reference

``` r
sibfldnm <- 'Derived_Data'
parent   <- dirname(getwd())
sibling  <- file.path(parent,sibfldnm)
```

## Load Excel Data

``` r
fn    <- 'Portland Cruise Ships Data.xlsx'
fpath <- file.path(sibling,fn)

the_data <- read_excel(fpath,
                       sheet = "Cruise", 
                      col_types = c("skip", "text", "date", 
                            "date", "date", "date", "text", "text", 
                            "text", "skip", "text", "skip", "text", 
                            "skip", "skip", "numeric", "numeric", 
                            "numeric", "numeric", "numeric", "skip")) %>%
  rename_all(~tolower(.)) %>%
  rename(vessel   = `vessel name`) %>%
  mutate(arr_date = as.Date(arr_date),
         dep_date = as.Date(dep_date)) %>%
  mutate(arr_time = as.numeric(format(arr_time, format = '%H')),
         arr_time = as.numeric(format(arr_time, format = '%H'))) %>%
  mutate(year     = as.numeric(format(arr_date, format = '%Y')),
         Month    = factor(as.numeric(format(arr_date, format = '%m')), levels = 1:12, labels = month.abb),
         doy      = as.numeric(format(arr_date, format = '%j')))

vessel_data <- read_excel(fpath, 
    sheet = "Vessels Pivot Table", col_types = c("text", 
        "numeric", "numeric", "numeric", 
        "numeric")) %>%
  rename(LOA = `Min of LOA`,
         BEAM = `Min of B`,
         DRAUGHT = `Min of D`,
         VISITS = `Count of ARR_DATE`)  %>%
  rename_all(~tolower(.))

nms <- names(vessel_data)
nms[1] <- 'vessel'
names(vessel_data) <- nms

ferry_data <-  read_excel(fpath, 
                          sheet = "Ferry", col_types = c("text", 
                              "date", "date", "date", "date", "text", 
                              "text", "text", "skip", "text")) %>%
  rename_all(~tolower(.)) %>%
  rename(vessel   = `vessel name`) %>%
  mutate(arr_date = as.Date(arr_date),
         dep_date = as.Date(dep_date)) %>%
  mutate(arr_time = as.numeric(format(arr_time, format = '%H')),
         arr_time = as.numeric(format(arr_time, format = '%H'))) %>%
  mutate(year     = as.numeric(format(arr_date, format = '%Y')),
         Month    = factor(as.numeric(format(arr_date, format = '%m')), levels = 1:12, labels = month.abb),
         doy      = as.numeric(format(arr_date, format = '%j')))
```

# Strategy

We want to be able to look at how visits to Portland are changing, so we
want:

  - Annual total number of visits

  - Annual total number of passengers

  - Annual size distribution of ships – both length and passengers

  - Annual pattern of monthly arrivals – ships

  - Annual pattern of monthly arrivals – passengers

It would be nice to look at crew numbers too, but that data is
incomplete, especially for 2017.

Other interesting questions: When do ships arrive and depart? How long
do they spend in Port?

# Annual Totals

## Table

``` r
annual_data <- the_data %>%
  select(-c(2:10)) %>%
  group_by(year) %>%
  summarize( visits       = sum(! is.na(vessel)),
             passengers   = sum(pax, na.rm = TRUE),
             meanpax      = mean(pax, na.rm=TRUE),
             medianpax    = median(pax, na.rm= TRUE),
             meanship     = mean(loa, na.rm=TRUE),
             medianship   = median(loa, na.rm=TRUE))
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

``` r
knitr::kable(annual_data,
             col.names = c('Year', 'Visits', 'Passengers', 'Mean Passenger #', 'Mean Ship Length (ft)', 'Median Passenger #', 'Median Ship Length (ft)'),
             digits = c(0,0,1,0,1,0))
```

| Year | Visits | Passengers | Mean Passenger \# | Mean Ship Length (ft) | Median Passenger \# | Median Ship Length (ft) |
| ---: | -----: | ---------: | ----------------: | --------------------: | ------------------: | ----------------------: |
| 2015 |     83 |      94408 |              1137 |                   468 |                 595 |                     610 |
| 2016 |     77 |     100546 |              1306 |                   913 |                 624 |                     715 |
| 2017 |     92 |     134643 |              1464 |                  1225 |                 638 |                     820 |
| 2018 |    110 |     157853 |              1448 |                   795 |                 641 |                     663 |
| 2019 |    106 |     157589 |              1487 |                  1805 |                 665 |                     856 |

# Graphics for visits and passengers

``` r
annual_data %>% 
  select(! contains('me')) %>%   # drop means and medians, which we don't want here.
  mutate(passengers = passengers/1000) %>%
  pivot_longer(-year, names_to = 'Metric', values_to = 'value') %>%

ggplot(aes(year, value, color=Metric)) + geom_line() + geom_point (shape = 17, size = 3) +
  scale_y_continuous(name = 'Cruise Ship Visits',
                     limits = c(0,175),
                     sec.axis = sec_axis(~ .*1000, name = 'Passengers',
                                         #labels = scales::unit_format(unit = 'Thousand', scale  = 1e-3))) +
                                         labels = scales::comma)) +
  scale_color_manual(values = cbep_colors(), labels = c('Passengers', 'Ships'),
                     name = '') +
  xlab('Year') +
  theme_cbep()
```

![](Analysis_Cruise_Ship_Data_files/figure-gfm/visits_graphic-1.png)<!-- -->

``` r
ggsave('annualvisits.pdf', device = cairo_pdf, width = 7, height = 5)
ggsave('annualvisits.png', type = 'cairo',   width = 7, height = 5)
```

# Graphics for Ship Size

``` r
annual_data %>% 
  select(year, contains('median')) %>%   # retain medians
  pivot_longer(-year, names_to = 'Metric', values_to = 'value') %>%

ggplot(aes(year, value, color=Metric)) + geom_line() + geom_point (shape = 17, size = 3) +
  ylab ("Median Size of Cruise Ships") + 
  ylim(c(0,2000)) +
  scale_color_manual(values = cbep_colors(), labels = c('Passemgers', 'Ship Length')) +
  xlab('Year') +
  theme_cbep()
```

![](Analysis_Cruise_Ship_Data_files/figure-gfm/ship_size_graphic-1.png)<!-- -->

``` r
#ggsave('shipsizes.pdf', device = cairo_pdf, width = 7, height = 5)
#ggsave('shipsizes.png', type = 'cairo',   width = 7, height = 5)
```

``` r
the_data %>% 
  select(arr_date, loa, pax) %>%

ggplot(aes(loa, pax))+
  geom_hex(aes(fill=stat(log2(count))), bins=15) +

  geom_smooth(se = FALSE, color='yellow') + 
  theme_cbep() +
  
  xlab('Ship Length') +
  ylab('Number of Passengers')
```

    ## Warning: Removed 1 rows containing non-finite values (stat_binhex).

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

    ## Warning: Removed 1 rows containing non-finite values (stat_smooth).

![](Analysis_Cruise_Ship_Data_files/figure-gfm/ship_size_graphic_density-1.png)<!-- -->
So, the two strongest clusters are for ships under 350 feet, with
typically under 500 passengers, and over 900 feet with over about 1500

Lets look at a cross tab like that.

``` r
loacat = cut(the_data$loa, breaks = c(0, 350, 900, 5000), labels = c("Small", "Medium", "Large"))
paxcat = cut(the_data$pax, breaks = c(0, 500, 1500, 5000), labels = c("Small", "Medium", "Large"))

table(loacat, paxcat)
```

    ##         paxcat
    ## loacat   Small Medium Large
    ##   Small    178      0     0
    ##   Medium    28     33    26
    ##   Large      0      0   195

We can refine the categories slightly to reduce cross-classification
errors.

``` r
loacat = cut(the_data$loa, breaks = c(0, 500, 800, 5000), labels = c("Small", "Medium", "Large"))
paxcat = cut(the_data$pax, breaks = c(0, 500, 1500, 5000), labels = c("Small", "Medium", "Large"))

table(loacat, paxcat)
```

    ##         paxcat
    ## loacat   Small Medium Large
    ##   Small    185      0     0
    ##   Medium    21     30     3
    ##   Large      0      3   218

What we see are the Silver Line ships are longer for the number of
passengers than most….

``` r
the_data %>% mutate(category = loacat) %>%

ggplot(aes(year, ..count.., fill = category) ) + geom_bar() +
  scale_fill_manual (values = cbep_colors2()[4:2],
                     labels = c('Small (< 500 ft)', 'Medium', 'Large (> 800 ft)'),
                     name = '') +
  theme_cbep() +
  ylab ("Number of Vessels") +
  xlab("Year") +
  ylim(0,120)
```

![](Analysis_Cruise_Ship_Data_files/figure-gfm/visits_barchart-1.png)<!-- -->

``` r
ggsave('visitsbysize.pdf', device = cairo_pdf, width = 7, height = 5)
ggsave('visitbysize.png', type = 'cairo',   width = 7, height = 5)
```
