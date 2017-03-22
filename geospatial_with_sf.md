-   [The `sf` package](#the-sf-package)
    -   [Exercise 1](#exercise-1)
-   [Reading in spatial data with
    `sf`](#reading-in-spatial-data-with-sf)
    -   [Simple Features](#simple-features)
    -   [Get some data to use](#get-some-data-to-use)
    -   [Read in POINT](#read-in-point)
    -   [Read in LINESTRING](#read-in-linestring)
    -   [Read in POLYGON](#read-in-polygon)
    -   [Performance](#performance)
    -   [Exercise 2](#exercise-2)
-   [Basics of `sf` objects](#basics-of-sf-objects)
    -   [Its a data.frame!](#its-a-data.frame)
    -   [Manipulate `sf` objects with `dplyr`, yes,
        `dplyr`!](#manipulate-sf-objects-with-dplyr-yes-dplyr)
    -   [Exercise 3](#exercise-3)
-   [Plotting](#plotting)
    -   [Base](#base)
    -   [Mapview](#mapview)
    -   [Exercise 4](#exercise-4)
-   [Analysis](#analysis)
    -   [Buffer and summarize](#buffer-and-summarize)
    -   [Exercise 5](#exercise-5)

In this mini-workshop we will introduce the `sf` package, show some
examples of geospatial analysis, work with base plotting of `sf`
objects, and show how `mapview` can be used to map these objects. It is
assumed that you have [R and RStudio
installed](https://github.com/rhodyrstats/intro_r_workshop#software) and
that you, at a minimum, understand the basic concepts of the R language
(e.g. you can work through[R For Cats](https://rforcats.net)).

Also as an aside, I am learning the `sf` package right now so, we will
be learning all of this together!

The `sf` package
================

Things are changing quickly in the R/spatial analysis world and the most
fundamental change is via the `sf` package. This package aims to replace
`sp`, `rgdal`, and `rgeos`. There are a lot of reasons why this is a
good thing, but that is a bit beyond the scope of this workshop Suffice
it to say it should make things faster and simpler!

To get started, lets get \`sf installed:

    install.packages("sf")
    library("sf")

It does rely on having access the GDAL, GEOS, and Proj.4 libraries. On
Windows and Mac this should be pretty straightforward.

Exercise 1
----------

The first exercise won't be too thrilling, but we need to make sure
everyone has the packages installed.

1.  Install `sf`.
2.  Load `sf`.
3.  If you don't have `dplyr` already, make sure it is installed.
4.  Load `dplyr`.

Reading in spatial data with `sf`
=================================

Simple Features
---------------

So, what does `sf` actually provide us? It is an implementation of an
ISO standard for storing spatial data. It forms the basis for many of
the common vector data models and is centered on the concept of a
"feature". Essentially a feature is any object in the real world. There
are many different types of features and there are different details
that get stored about each. For details on this the [first `sf`
vignette](https://cran.r-project.org/web/packages/sf/vignettes/sf1.html)
does a really nice job. For this mini-workshop we are going to focus on
three feature types, POINT, MULTILINESTRING, and MULTIPOLYGON.

For each of the types, there will be coordinates stored as dimensions, a
coordinate reference system, and attributes.

Get some data to use
--------------------

We can grab some data directly from the Rhode Island Geographic
Information System (RIGIS) for these examples.

    # Municipal Boundaries
    download.file(url = "http://www.rigis.org/geodata/bnd/muni97d.zip",
                  destfile = "data/muni97d.zip")
    unzip(zipfile = "data/muni97d.zip", 
          exdir = "data")

    # Streams
    download.file(url = "http://www.rigis.org/geodata/hydro/streams.zip",
                  destfile = "data/streams.zip")
    unzip(zipfile = "data/streams.zip", 
          exdir = "data")

    # Potential Growth Centers
    download.file(url = "http://www.rigis.org/geodata/plan/growth06.zip",
                  destfile = "data/growth06.zip")
    unzip(zipfile = "data/growth06.zip", 
          exdir = "data")

    # Land Use/Land Cover
    download.file(url = "http://www.rigis.org/geodata/plan/rilc11d.zip",
                  destfile = "data/rilc11d.zip")
    unzip(zipfile = "data/rilc11d.zip", 
          exdir = "data")

Read in POINT
-------------

    growth_cent <- st_read("data/growth06.shp")

    ## Reading layer `growth06' from data source `/data/jhollist/geospatial_with_sf/data/growth06.shp' using driver `ESRI Shapefile'
    ## Simple feature collection with 21 features and 2 fields
    ## geometry type:  POINT
    ## dimension:      XY
    ## bbox:           xmin: 260137.3 ymin: 32916.7 xmax: 418116.3 ymax: 326549.2
    ## epsg (SRID):    NA
    ## proj4string:    +proj=tmerc +lat_0=41.08333333333334 +lon_0=-71.5 +k=0.99999375 +x_0=99999.99999999999 +y_0=0 +datum=NAD83 +units=us-ft +no_defs

Read in LINESTRING
------------------

    streams <- st_read("data/streams.shp")

    ## Reading layer `streams' from data source `/data/jhollist/geospatial_with_sf/data/streams.shp' using driver `ESRI Shapefile'
    ## Simple feature collection with 4470 features and 8 fields
    ## geometry type:  LINESTRING
    ## dimension:      XY
    ## bbox:           xmin: 234010.1 ymin: 31361.37 xmax: 430921.9 ymax: 340865.8
    ## epsg (SRID):    NA
    ## proj4string:    +proj=tmerc +lat_0=41.08333333333334 +lon_0=-71.5 +k=0.99999375 +x_0=99999.99999999999 +y_0=0 +datum=NAD83 +units=us-ft +no_defs

Read in POLYGON
---------------

    muni <- st_read("data/muni97d.shp")

    ## Reading layer `muni97d' from data source `/data/jhollist/geospatial_with_sf/data/muni97d.shp' using driver `ESRI Shapefile'
    ## Simple feature collection with 396 features and 12 fields
    ## geometry type:  POLYGON
    ## dimension:      XY
    ## bbox:           xmin: 220310.4 ymin: 23048.49 xmax: 432040.9 ymax: 340916.6
    ## epsg (SRID):    NA
    ## proj4string:    +proj=tmerc +lat_0=41.08333333333334 +lon_0=-71.5 +k=0.99999375 +x_0=99999.99999999999 +y_0=0 +datum=NAD83 +units=us-ft +no_defs

Performance
-----------

One of the benefits of using `sf` is the speed. In my tests it is about
twice as fast. Let's look at a biggish shape file with 1 million points!

![1 million
points](https://media4.giphy.com/media/13B1WmJg7HwjGU/giphy.gif)

    #The old way
    system.time(readOGR("data","big"))

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "data", layer: "big"
    ## with 1000000 features
    ## It has 1 fields

    ##    user  system elapsed 
    ##  11.093   0.597  11.722

    #The sf way
    system.time(st_read("data/big.shp"))

    ## Reading layer `big' from data source `/data/jhollist/geospatial_with_sf/data/big.shp' using driver `ESRI Shapefile'
    ## Simple feature collection with 1000000 features and 1 field
    ## geometry type:  POINT
    ## dimension:      XY
    ## bbox:           xmin: -71.03768 ymin: 41.05976 xmax: -69.09763 ymax: 43.00856
    ## epsg (SRID):    4326
    ## proj4string:    +proj=longlat +datum=WGS84 +no_defs

    ##    user  system elapsed 
    ##   5.419   0.069   5.492

Exercise 2
----------

1.  Read in shapefiles with

Basics of `sf` objects
======================

Its a data.frame!
-----------------

Manipulate `sf` objects with `dplyr`, yes, `dplyr`!
---------------------------------------------------

Exercise 3
----------

Plotting
========

Base
----

Mapview
-------

Exercise 4
----------

Analysis
========

Buffer and summarize
--------------------

Exercise 5
----------
