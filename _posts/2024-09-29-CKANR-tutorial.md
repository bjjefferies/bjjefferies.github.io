---
layout: post
title: "CKANR API Tutorial"
output:
  md_document:
    variant: markdown_github
    preserve_yaml: true
    toc: yes
---

-   [CKAN R API](#ckan-r-api)
    -   [Packages](#packages)
    -   [Creating Your Account](#creating-your-account)
    -   [Connect to the PHX open data
        API](#connect-to-the-phx-open-data-api)
        -   [Pull Data from the API (Naloxone
            Distribution)](#pull-data-from-the-api-naloxone-distribution)
        -   [Pull Data from the API
            (Overdose)](#pull-data-from-the-api-overdose)
    -   [Visualize Data](#visualize-data)

# CKAN R API

CKAN stands for Comprehensive Knowledge Archive Network and it is a
popular tool for municipalities that wish to share their data publicly.
Many governmental agencies in the US and across the globe utilize CKAN,
and in this tutorial we will be accessing the City of Phoenix CKAN open
data and exploring a data set.

The best part about the way we pull in data with the API in this
tutorial is that every time we re-run the code in this script, the data
will automatically update from the website.

<br>

## Packages

You will need to download the ckanr package, directions can be followed
[here](https://github.com/ropensci/ckanr). The remaining packages are
listed in the code below

``` r
library(ckanr)
library(tidyverse)
library(curl)
library(lubridate)
library(pander)
```

<br>

## Creating Your Account

Most API’s have some level of security around them. You will need to
register for an API Key at <https://www.phoenixopendata.com/>. This can
be a slightly tricky process and you may need to search for videos on
how to register for an API with ckan. Eventually, you will see your API
key in your profile on the phoenixopendata.com site.

Format of API key: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

``` r
knitr::include_graphics("phx_open_API.png")
```

<img src="{{site.url}}/assets/img/phx_open_API.png" width="400px" style="display: block; margin: auto;" />

<br>

## Connect to the PHX open data API

To connect, you will need to run the ckanr_setup() function as below. I
saved my personal key as the variable my.key so that you couldn’t see it
in this tutorial.

To check that our connection is successful run the group_list call which
will show the various data groups established within the PHX open data
site. CKAN databases are always grouped, which can help you navigate
towards the data you are looking for.

``` r
ckanr::ckanr_setup(url = "https://www.phoenixopendata.com/",
                   key = my.key)

ckanr::group_list(as = "table")
```

    ##  [1] "external-datasets"        "transportation"          
    ##  [3] "city-performance-metrics" "cityservices"            
    ##  [5] "finance"                  "mapping"                 
    ##  [7] "public-safety"            "transportation1"         
    ##  [9] "energy-sustainability"    "water"

<br>

### Pull Data from the API (Naloxone Distribution)

Ultimately, we are interested in Resources, which are the actual
datasets within a CKAN database structure. We can search for resources
in the following way. I am interested in getting datasets on naloxone
and on overdoses.

Notice that in the results we see the <CKAN Resource> id. We can use
those id numbers to pull our desired data set.

``` r
ckanr::resource_search("name:naloxone")
```

    ## $count
    ## [1] 2
    ## 
    ## $results
    ## $results[[1]]
    ## <CKAN Resource> ac064a1f-5653-4dc7-834e-b133982b49be 
    ##   Name: Naloxone Administered Prior to EMS Arrival
    ##   Description: 
    ##   Creator/Modified: 2024-05-03T16:10:36.450459 / 2024-09-05T14:10:45.085782
    ##   Size: 2559
    ##   Format: CSV
    ## 
    ## $results[[2]]
    ## <CKAN Resource> 4e8e45f3-6841-4387-98e5-2b68f1e07682 
    ##   Name: Naloxone Distributed by Month
    ##   Description: 
    ##   Creator/Modified: 2024-05-03T17:09:49.284415 / 2024-09-05T14:11:22.189423
    ##   Size: 14595
    ##   Format: CSV

<br>

We will be looking at the monthly distribution of Naloxone in the City
of Phoenix. I copy/pasted the resource id into the resource_show
function and saved the results as x. x is now a list with a lot of
information, including the URL of the dataset contained in this
resource.

``` r
x <- ckanr::resource_show(id = "4e8e45f3-6841-4387-98e5-2b68f1e07682")
str(x)
```

    ## List of 27
    ##  $ cache_last_updated                           : NULL
    ##  $ cache_url                                    : NULL
    ##  $ ckan_url                                     : chr "https://www.phoenixopendata.com"
    ##  $ created                                      : chr "2024-05-03T17:09:49.284415"
    ##  $ datastore_active                             : logi TRUE
    ##  $ datastore_contains_all_records_of_source_file: logi TRUE
    ##  $ description                                  : chr ""
    ##  $ format                                       : chr "CSV"
    ##  $ hash                                         : chr "3e99aeec3240c47910df3280848b372d"
    ##  $ id                                           : chr "4e8e45f3-6841-4387-98e5-2b68f1e07682"
    ##  $ ignore_hash                                  : logi FALSE
    ##  $ last_modified                                : chr "2024-09-05T14:11:22.189423"
    ##  $ metadata_modified                            : chr "2024-09-05T14:11:22.207637"
    ##  $ mimetype                                     : NULL
    ##  $ mimetype_inner                               : NULL
    ##  $ name                                         : chr "Naloxone Distributed by Month"
    ##  $ original_url                                 : chr "https://www.phoenixopendata.com/dataset/449acb7c-90aa-43c9-b5b4-d0b2b8e44a36/resource/4e8e45f3-6841-4387-98e5-2"| __truncated__
    ##  $ package_id                                   : chr "449acb7c-90aa-43c9-b5b4-d0b2b8e44a36"
    ##  $ position                                     : int 0
    ##  $ resource_id                                  : chr "4e8e45f3-6841-4387-98e5-2b68f1e07682"
    ##  $ resource_type                                : NULL
    ##  $ set_url_type                                 : logi FALSE
    ##  $ size                                         : int 14595
    ##  $ state                                        : chr "active"
    ##  $ task_created                                 : chr "2024-09-05 14:11:22.338129"
    ##  $ url                                          : chr "https://www.phoenixopendata.com/dataset/449acb7c-90aa-43c9-b5b4-d0b2b8e44a36/resource/4e8e45f3-6841-4387-98e5-2"| __truncated__
    ##  $ url_type                                     : chr "upload"
    ##  - attr(*, "class")= chr "ckan_resource"

<br>

Notice when we look specifically at the url of the resource it is a .csv
file. We can read that csv file into R with the following code.

``` r
nalox.dist <- read.csv(curl(x$url))
```

<br>

Now we can clean our data so that we can easily aggregate the amount of
naloxone distributed each month (since data started being collected in
July 2023)

``` r
#covert char dates to posixCT
nalox.dist$DATE <- lubridate::mdy(nalox.dist$PERIOD) 

# Create column for year
nalox.dist$YEAR <- lubridate::year(nalox.dist$DATE)   

#convert Month to numeric
nalox.dist$MON <- gsub(" - [A-Z]+", "", nalox.dist$MONTH) %>% as.numeric() 

nalox.dist %>% 
  group_by(YEAR, MON) %>% 
  summarise(units.dist = sum(TAKE_HOME)) %>% 
  pander()
```

| YEAR | MON | units.dist |
|:----:|:---:|:----------:|
| 2023 |  7  |     0      |
| 2023 |  8  |    1394    |
| 2023 |  9  |    455     |
| 2023 | 10  |    578     |
| 2023 | 11  |    384     |
| 2023 | 12  |    693     |
| 2024 |  1  |    463     |
| 2024 |  2  |    735     |
| 2024 |  3  |    1035    |
| 2024 |  4  |    1198    |
| 2024 |  5  |    1070    |
| 2024 |  6  |    956     |
| 2024 |  7  |    1337    |

<br>

### Pull Data from the API (Overdose)

To supplement the data on naloxone distribution, we are also going to
pull data on overdoses. I first search for datasets with overdose and I
found an interesting one called “Suspected Opioid Overdoses by Month”. I
copied the id and read in the data. However, that dataset only contains
data until the end of 2023. After some searching I realized phx is
keeping the current 2024 data in a different file “Suspected Opioid
Overdoses by Year to Date”.

Fortunately both data sets have the same column format, so we can simply
rbind the 2024 (y) data set to the (x) data set.

``` r
ckanr::resource_search("name:overdose", limit = 2) #take away limit to see all of the resources
```

    ## $count
    ## [1] 15
    ## 
    ## $results
    ## $results[[1]]
    ## <CKAN Resource> 3c029e52-8787-4d09-9637-ac21b2c0dab9 
    ##   Name: Fatal Overdoses by Location
    ##   Description: 
    ##   Creator/Modified: 2024-05-03T15:52:46.089491 / 2024-09-05T14:10:29.026004
    ##   Size: 1949
    ##   Format: CSV
    ## 
    ## $results[[2]]
    ## <CKAN Resource> b65b5a9c-9f06-4c57-9d7a-ee3451d4ce0b 
    ##   Name: Fatal Overdoses by Race Ethnicity
    ##   Description: 
    ##   Creator/Modified: 2024-05-03T15:57:14.242857 / 2024-09-05T14:10:40.085834
    ##   Size: 585
    ##   Format: CSV

``` r
x <- ckanr::resource_show(id = "841c6d87-3d9c-4e21-a584-af252679402a")
y <- ckanr::resource_show(id = "99cbb95c-590e-4474-b774-362202c66367")

x <- read.csv(curl(url = x$url))

y <- read.csv(curl(url = y$url))

opioid.od <- rbind(x, y)

nrow(x) + nrow(y) == nrow(opioid.od) #Check to ensure no lost data
```

    ## [1] TRUE

<br>

I also need to clean this data up so that we can see it listed out by
month and year.

``` r
opioid.od$DATE <- lubridate::mdy(opioid.od$PERIOD)
opioid.od$YEAR <- lubridate::year(opioid.od$DATE)
opioid.od$MON <- lubridate::month(opioid.od$DATE)

opioid.od %>% 
  filter(YEAR >= 2023) %>% 
  select(YEAR, MON, COUNT)
```

    ##    YEAR MON COUNT
    ## 1  2023   1   332
    ## 2  2023   2   286
    ## 3  2023   3   378
    ## 4  2023   4   377
    ## 5  2023   5   416
    ## 6  2023   6   376
    ## 7  2023   7   536
    ## 8  2023   8   427
    ## 9  2023   9   372
    ## 10 2023  10   380
    ## 11 2023  11   338
    ## 12 2023  12   336
    ## 13 2024   1   310
    ## 14 2024   2   286
    ## 15 2024   3   301
    ## 16 2024   4   364
    ## 17 2024   5   320
    ## 18 2024   6   348
    ## 19 2024   7   419
    ## 20 2024   8   399

## Visualize Data

We can now produce some quick visualizations to see if there is any
connection between naloxone distribution and suspected opioid overdoses
in phoenix.

``` r
opioid.od$year_month <- paste0(year(opioid.od$DATE), "-", sprintf("%02d", month(opioid.od$DATE)))

# factor and level year_month for graph purposes
opioid.od$year_month <- factor(opioid.od$year_month, levels = unique(opioid.od$year_month)[order(opioid.od$DATE)])

opioid.od %>% 
  filter(YEAR >= 2023) %>% 
  ggplot(aes(x = year_month, y = COUNT)) +
  geom_col(fill = "darkblue") +
  labs(title = "Suspected Opioid OD by Month",
       subtitle = "City Of Phoenix, AZ",
       y = "Suspected Overdoses",
       x = "Month and Year") +
  theme(axis.text.x = element_text(angle = 90, hjust = 1, vjust = .3),
        panel.grid.major.x = element_blank())
```

<img src="{{site.url}}/assets/img/opioid.od.graph-1.png" style="display: block; margin: auto;" />

<br>

And we can compare that to Naloxone distribution rates.

``` r
nalox.dist$year_month <- paste0(year(nalox.dist$DATE), "-", sprintf("%02d", month(nalox.dist$DATE)))

nalox.dist.t <- nalox.dist %>% 
  group_by(year_month) %>% 
  summarise(count = sum(TAKE_HOME))

# Add blank rows to the top so x axis matches overdose graph
extra.dates <- c("2023-01", "2023-02", "2023-03", "2023-04", "2023-05", "2023-06")
extra.count <- rep(0,6)
extra.rows <- data.frame(year_month = extra.dates, count = extra.count)

nalox.dist.t <- rbind(nalox.dist.t, extra.rows)

nalox.dist.t %>% 
  ggplot(aes(x = year_month, y = count)) +
  geom_col(fill = "orange") +
  theme(axis.text.x = element_text(angle = 90, hjust = 1, vjust = 0.3),
        panel.grid.major.x = element_blank()) +
  labs(title = "Distributed Naloxone in Phoenix, AZ",
       subtitle = "Distribution began in August, 2023",
       y = "Units of Naloxone Distributed",
       x = "Month and Year")
```

<img src="{{site.url}}/assets/img/nalox.dist.graph-1.png" style="display: block; margin: auto;" />
