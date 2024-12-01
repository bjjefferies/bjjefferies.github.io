---
layout: post
title: "GIS Data in R - Advanced Features"
output:
  md_document:
    variant: markdown_github
    preserve_yaml: true
    toc: yes
---

-   [Advanced Techniques with Spatial
    Data](#advanced-techniques-with-spatial-data)
    -   [Load Packages and Data](#load-packages-and-data)
        -   [geometry](#geometry)
    -   [plotting with ggplot](#plotting-with-ggplot)
    -   [st_centroid()](#st_centroid)
    -   [st_distance()](#st_distance)
    -   [Visualize the distance](#visualize-the-distance)
-   [Further Resources](#further-resources)
-   [Works Cited](#works-cited)

# Advanced Techniques with Spatial Data

This code-through aims to enhance the readers understanding of some of
the advanced tools one can utilize while working with spatial data in R.
Specfically, we will focus on the [sf
package](https://cran.r-project.org/web/packages/sf/index.html). sf is a
package that deals with “simple features” which is a common and
standardized way of working with spatial data, similar to Spatial
Polygon data but offering a bit more flexibility.

In this tutorial, I will walk through using sf data structures and
functions in order to recreate the engineering of one of the variables
used in Dr. Ken Steif’s study “Predicting gentrification using
longitudinal census data” [Urban
Spatial](https://urbanspatialanalysis.com/portfolio/predicting-gentrification-using-longitudinal-census-data/).
The variable of interest is a measurement of the distance, in meters, of
any urban tract to the nearest “wealthy” tract, meaning it falls in the
upper 20% of median home values in that urban area. We will use data for
Chicago to create that variable.

<br>

## Load Packages and Data

The following packages are necessary to follow along. If you have not
installed them, you will need to run install.packages(“packagename”).

``` r
library(sf)
library(tidyverse)
library(ggplot2)
library(geojsonio)
library(geojsonsf)
library(gtools)


# to see help documentation for any of the functions used type ?function()
# for example:
# ?quantcut()
```

<br>

To load the data for this tutorial, you can download a pre-prepared file
from github with the following code.

``` r
# url with the data
url <- "https://raw.githubusercontent.com/bjjefferies/MSPEDA/refs/heads/main/PAF_516/chi_sf.geojson"

#read in the geojson file.  what = "sp" stands for spatial polygon object
chi_sf <- geojson_read(url, what = "sp")

#convert sp to sf object
chi_sf <- sf::st_as_sf(chi_sf)
```

<br>

For this analysis we also need to determine which census tracts are in
the upper 20% (5th quintile) of median home value.

``` r
# create quintiles from mhmval12 and assign highest to new binary variable - q5
chi_sf$q5 <- quantcut(chi_sf$mhmval12, q = 5) #cuts data into quintiles
levels(chi_sf$q5) <- c(1,2,3,4,5) # assign 1-5 to quintiles
chi_sf$q5 <- as.numeric(chi_sf$q5) #convert from factor to numeric
chi_sf$q5 <- ifelse(chi_sf$q5 == 5, 1, 0) #replace all quintiles with 0, but 5
```

<br>

### geometry

In all sf files, a column, or variable, is created called “geometry”. It
stores the geo-spatial data of the file. The output below shows that
this file is storing polygons, the set of points that bound each census
tract.

``` r
head(chi_sf$geometry, 3)
```

    ## Geometry set for 3 features 
    ## Geometry type: POLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: -87.68465 ymin: 42.01232 xmax: -87.66434 ymax: 42.02313
    ## Geodetic CRS:  WGS 84

<br>

## plotting with ggplot

Below is an example how easy it is to plot sf files with ggplot2. The
geom_sf() automatically finds the geometry variable within the data file
passed to ggplot and takes that data to draw the outlines of each
polygon, which in this case represents census tracts. The fill shows the
median home value in each tract in the year 2012.

``` r
ggplot(data = chi_sf) +
  geom_sf(aes(fill = mhmval12), colour = NA)
```

![]({{site.url}}/assets/img/sf-image-1-1.png)

<br>

## st_centroid()

For this investigation we want to know the center point of each tract so
we can use those points to measure distance between each tract. The
st_centroid() function is used below to find the centroid of any
polygon. It returns a point with two coordinates (longitude, latitude).

``` r
# find center point or centroid of the first census tract
st_centroid(chi_sf$geometry[1])
```

    ## Geometry set for 1 feature 
    ## Geometry type: POINT
    ## Dimension:     XY
    ## Bounding box:  xmin: -87.67021 ymin: 42.02124 xmax: -87.67021 ymax: 42.02124
    ## Geodetic CRS:  WGS 84

To find all of the centroids in our data set, one for each tract, we can
run the following code. I also plot the centroids in order to visualize.

``` r
# to plot points, we need x and y coordinates, longitude and latitude 
chi_sf <- chi_sf %>% 
  #first create spatial object of the centroid
  mutate(centroid = st_centroid(geometry)) %>% 
  # st_coordinates converts spatial object to matrix, we extract the columns
  mutate(cent.long = st_coordinates(centroid)[ ,1],
         cent.lat = st_coordinates(centroid)[ ,2])

#plot to show it worked
ggplot(data = chi_sf) +
  geom_sf(aes(fill = mhmval12)) +
  geom_point(aes(x = cent.long, y = cent.lat), color = "black", cex = .5)
```

![]({{site.url}}/assets/img/sf-image-2-1.png)

<br>

## st_distance()

Below is a function which I created, with the help of Chat-GPT, to find
the minimum distance between each tract and it’s nearest top-quintile
neighbor. The distances are reported in meters.

``` r
# Function to find minimum distance to top quintile centroids
calculate_minimum_distances <- function(data, target_rows) {
  
  target_centroids <- data[target_rows, "centroid", drop = FALSE]
  
  # Initialize a vector to store minimum distances
  min_distances <- numeric(nrow(data))
  
  # Calculate distances for each row
  for (i in seq_len(nrow(data))) {
    # Calculate all distances from the i-th centroid to target centroids
    distances <- st_distance(data$centroid[i], target_centroids)
    
    # Find the minimum distance and store it
    min_distances[i] <- min(distances)
  }
  
  return(min_distances)
}
```

This code uses the function and stores the data as the variable
“distance” in the data frame. The output is in meters, so I multiply by

``` r
chi_sf$distance <- calculate_minimum_distances(data = chi_sf, which(chi_sf$q5 == 1))
```

## Visualize the distance

We can visualize the distance data that we now have, but it is
relatively meaningless as presented. The far suburban neighborhoods are
light colored, but because of the spread and scale of the distances,
nearly all of the nearly 2000 tracts represented appear dark colored.

``` r
ggplot(chi_sf) +
  geom_sf(aes(fill = distance), color = NA)
```

![]({{site.url}}/assets/img/sf-image-3-1.png)

<br>

The summary below demonstrates the scale issue. The median distance is
1600 meters, but the mean is way higher, meaning there are some large
outliers in our data.

``` r
summary(chi_sf$distance)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##     0.0   362.8  1600.0  3705.0  4455.6 51131.6

<br>

We can correct this by re-scaling our distance data and applying a more
appropriate color gradient scheme. As shown in the histograms below, we
can achieve a much more normal distribution of the data with a logarithm
transformation. So, we will take the *l**o**g*<sub>10</sub> of our
distance data, store that as a new variable, and plot it again.

In the plot below, the gray areas are the tracts that are in the top
quintile of median home values, and one can see the changes in color
gradient getting lighter as tracts are further from a top quintile
tract. It is important to note that the raw data on distance, before log
transformation, is the data used in producing the statistical models in
the Steif study upon which this article draws inspiration, not the log
transformed data which is only useful for visuals.

``` r
par(mfrow = c(1,2))
hist(chi_sf$distance)
hist(log(chi_sf$distance))
```

![]({{site.url}}/assets/img/sf-image-4-1.png)

``` r
chi_sf$distance.log <- log(chi_sf$distance)

ggplot(chi_sf) +
  geom_sf(aes(fill = distance.log), color = NA) +
  scale_fill_fermenter() +
  theme(panel.background = element_blank())
```

![]({{site.url}}/assets/img/sf-image-5-1.png)

# Further Resources

1.  [sf package github](https://r-spatial.github.io/sf/)

-   Complete vignettes by the sf package maintainers.

1.  Spatial R: Using the sf package \| UVA Library. (n.d.).
    <https://library.virginia.edu/data/articles/spatial-r-using-sf-package>

-   Another great walkthrough that goes into the very basics of spatial
    objects in R

# Works Cited

1.  Steif, K., Mallach, A., Fichman, M., & Kassel, S. (n.d.). Predicting
    gentrification using longitudinal census data. Urban Spatial.
    Retrieved from
    <https://urbanspatialanalysis.com/portfolio/predicting-gentrification-using-longitudinal-census-data/>
