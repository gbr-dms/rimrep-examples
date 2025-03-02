Extracting gridded data via the API
================
Denisse Fierro Arcos
2024-05-20

- [Extracting sea surface temperature from NOAA Coral Reef
  Watch](#extracting-sea-surface-temperature-from-noaa-coral-reef-watch)
  - [Loading libraries](#loading-libraries)
  - [Loading Whitsunday features from GBR
    shapefile](#loading-whitsunday-features-from-gbr-shapefile)
  - [Connecting to RIMREP collection via
    API](#connecting-to-rimrep-collection-via-api)
  - [Plotting data](#plotting-data)
  - [Calculating monthly sea surface temperature
    mean](#calculating-monthly-sea-surface-temperature-mean)
  - [Calculating time series and maximum monthly
    values](#calculating-time-series-and-maximum-monthly-values)
- [Extra examples during workshop](#extra-examples-during-workshop)

# Extracting sea surface temperature from NOAA Coral Reef Watch

The [NOAA Coral Reef Watch Sea Surface Temperature (CRW
SST)](https://coralreefwatch.noaa.gov/) is a high resolution (5 km)
dataset that provides daily SST data for coral reef monitoring. This
dataset is available in the DMS under the [**NOAA Coral Reef Watch**
collection](https://stac.reefdata.io/browser/collections/noaa-crw). In
this notebook, we will use access this dataset using the API. We will
also show how to extract data for a specific area using a bounding box
derived from a shapefile. For this step, we will use one of the
geometries available in the GBR Complete Features shapefile, which is
also available in the
[DMS](https://stac.reefdata.io/browser/collections/gbrmpa-admin-regions/items/gbrmpa-complete-gbr-features).

## Loading libraries

``` r
#Loading useful_functions script
source("useful_functions.R")
#Gridded data
library(terra)
library(tidyterra)
#Shapefiles
library(sf)
#Base maps
library(rnaturalearth)
#Data manupulation
library(tibble)
library(tidyr)
```

## Loading Whitsunday features from GBR shapefile

We will use the GBR features here, but we will only load boundaries for
the Whitsunday for this exercise. However, the function loading these
GBR features can load the entire shapefile or any other GBR feature.

``` r
#Loading boundaries for Whitsundays
whitsundays <- gbr_features(site_name = "Whitsunday")
```

    ## Subsetting GBR features by Whitsunday

``` r
#Checking results of query
whitsundays
```

    ## Simple feature collection with 42 features and 7 fields
    ## Geometry type: POLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 148.9111 ymin: -20.32891 xmax: 149.0637 ymax: -20.15519
    ## Geodetic CRS:  GDA94
    ## # A tibble: 42 × 8
    ##    UNIQUE_ID   GBR_NAME   LOC_NAME_S                  geometry FEAT_NAME LEVEL_1
    ##  * <chr>       <chr>      <chr>                  <POLYGON [°]> <chr>     <chr>  
    ##  1 20041111104 Whitsunda… Whitsunda… ((148.9313 -20.29419, 14… Reef      Reef   
    ##  2 20041124104 Whitsunda… Whitsunda… ((149.0355 -20.30176, 14… Reef      Reef   
    ##  3 20041103104 Whitsunda… Whitsunda… ((148.9969 -20.23033, 14… Reef      Reef   
    ##  4 20041121104 Whitsunda… Whitsunda… ((148.9929 -20.30349, 14… Reef      Reef   
    ##  5 20041119104 Whitsunda… Whitsunda… ((148.9694 -20.32657, 14… Reef      Reef   
    ##  6 20041118104 Whitsunda… Whitsunda… ((148.9614 -20.15612, 14… Reef      Reef   
    ##  7 20041113104 Whitsunda… Whitsunda… ((148.933 -20.26951, 148… Reef      Reef   
    ##  8 20041108104 Whitsunda… Whitsunda… ((148.9859 -20.31167, 14… Reef      Reef   
    ##  9 20041109104 Whitsunda… Whitsunda… ((148.9579 -20.30375, 14… Reef      Reef   
    ## 10 20041104104 Whitsunda… Whitsunda… ((149.0111 -20.23419, 14… Reef      Reef   
    ## # ℹ 32 more rows
    ## # ℹ 2 more variables: LEVEL_2 <chr>, LEVEL_3 <chr>

## Connecting to RIMREP collection via API

From the STAC catalogue item for the [NOAA CRW
SST](https://stac.reefdata.io/browser/collections/noaa-crw/items/noaa-crw-chs-sst?.language=en-AU),
we can get the link to the API from the *Additional Resources* section
of the page on the left under the map.

**Note:** Before running the code chunk below, make sure you either have
store your user credentials as environmental variables, or have this
information with you to input in the `connect_dms_dataset` function
below. Alternatively, if you already have an access token, you can
provide this as an input in the `connect_dms_dataset` function. Refer to
**The data API** subsection under **How to use DMS services and data**
in the [README
page](https://github.com/gbr-dms/rimrep-training/blob/main/CoTS-training-Jan2024/README.md)
for more information.

If you do not user credentials, you will not be able to access our API,
please contact the DMS team to set up an account by emailing
<rimrep-dms@aims.gov.au>.

``` r
#Defining API URL (obtained from STAC catalogue)
base_url <- "https://pygeoapi.reefdata.io/collections/noaa-crw-chs-sst/"

#Defining variable of interest (obtained from STAC catalogue)
variable_name <- "analysed_sst"

#Connecting to DMS and downloading data
sst_gbr <- connect_dms_dataset(base_url, variable_name, 
                               start_time = '1994-01-01', 
                               end_time = '2023-12-31',
                               bounding_shape = whitsundays)
```

    ## Warning: No 'access_token' and no user credentials were provided as input.

    ## Checking if 'CLIENT_ID' variable exists.

    ## Warning: No 'access_token' and user credentials were provided as input.

    ## Checking if 'CLIENT_SECRET' variable exists.

    ## Access token retrieved successfully.

## Plotting data

We will plot the first layer of this raster in a map to check the
temperature data.

``` r
#Get map of Australia
aust <- ne_countries(country = "Australia", returnclass = "sf")

#Start a plot
ggplot()+
  #Plot one raster layer
  geom_spatraster(data = sst_gbr$analysed_sst_1)+
  #Choose a nicer palette for our map
  scale_fill_distiller(palette = "YlOrRd", name = "SST")+
  #Add Australia
  geom_sf(data = aust)+
  #Add Whitsundays
  geom_sf(data = whitsundays, fill = NA, colour = "black")+
  #Establish map limits
  lims(x = c(148, 150), y = c(-20.5, -20))+
  #Apply a nice predefined theme
  theme_bw()+
  #Add a title
  labs(title = "Sea Surface Temperature")+
  #Center the plot title
  theme(plot.title = element_text(hjust = 0.5))
```

![](07-API_Data_Access_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

The grey areas are land areas without any temperature data.

## Calculating monthly sea surface temperature mean

We will calculate the monthly mean of the sea surface temperature for
the reefs surrounding the Whitsunday Island. We will use the
`raster_calc` function included in the `useful_functions.R` script. The
result will be gridded data.

``` r
#Applying function to calculate monthly mean
sst_mean <- raster_calc(sst_gbr, "monthly", "mean", na.rm = T)
```

    ## Multiple formats matched: "%Y-%Om"(1), "%Y-%m"(1)

    ## Using: "%Y-%Om"

``` r
#Checking result
sst_mean
```

    ## class       : SpatRaster 
    ## dimensions  : 4, 3, 360  (nrow, ncol, nlyr)
    ## resolution  : 0.04999542, 0.05000051  (x, y)
    ## extent      : 148.9, 149.05, -20.35, -20.15  (xmin, xmax, ymin, ymax)
    ## coord. ref. : +proj=longlat +a=6378137 +rf=298.257202148437 +no_defs 
    ## source(s)   : memory
    ## varname     : mean_analysed_sst 
    ## names       :    mean_1994-01,    mean_1994-02,    mean_1994-03,    mean_1994-04,    mean_1994-05,    mean_1994-06, ... 
    ## min values  :        28.54484,        28.30393,        27.20000,        26.03300,        24.09613,        22.19967, ... 
    ## max values  :        28.62742,        28.44179,        27.37581,        26.42633,        24.52323,        22.64300, ... 
    ## unit        : degrees_Celsius, degrees_Celsius, degrees_Celsius, degrees_Celsius, degrees_Celsius, degrees_Celsius, ... 
    ## time (days) : 1994-01-01 to 2023-12-01

We can see that the number of layers in the raster was reduced from
1.0957^{4} in the original raster downloaded from the API to 360 when
the monthly means were calculated.

We can now plot the monthly mean for the sea surface temperature. We
will do this for the first layer.

``` r
#Start a plot
ggplot()+
  #Plot one raster layer
  geom_spatraster(data = sst_mean$`mean_2019-01`)+
  #Choose a nicer palette for our map
  scale_fill_distiller(palette = "YlOrRd", name = "mean SST")+
  #Add Australia
  geom_sf(data = aust)+
  #Add Whitsundays
  geom_sf(data = whitsundays, fill = NA, colour = "black")+
  #Establish map limits
  lims(x = c(148, 150), y = c(-20.5, -20))+
  #Apply a nice predefined theme
  theme_bw()+
  #Add a title
  labs(title = "Mean SST")+
  #Center the plot title
  theme(plot.title = element_text(hjust = 0.5))
```

![](07-API_Data_Access_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

## Calculating time series and maximum monthly values

We can calculate the mean for each time step included in the raster to
obtain a time series (i.e., a single value for each layer). Then we can
identify the maximum value per month. The `ras_to_ts` function in the
`useful_functions.R` script will do this for us. This function returns a
list that includes the time series and the maximum monthly values from
the time series.

``` r
#Calculating time series and maximum monthly values
sst_ts <- ras_to_ts(sst_mean, mean, na.rm = T) 

#Plotting time series
sst_ts$time_series |> 
  ggplot(aes(x = time, y = mean))+
  geom_line()+
  scale_x_date(date_breaks = "2 year", date_labels = "%Y-%m",
               date_minor_breaks = "6 month")+
  theme_bw()+
  labs(title = "Monthly SST mean over Whitsunday Island",
       y = "Sea Surface Temperature (°C)")+
  theme(axis.title.x = element_blank(), 
        axis.text.x = element_text(angle = 45, hjust = 1))
```

![](07-API_Data_Access_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

We can also plot the maximum monthly values. This function could be used
to calculate the Maximum Monthly Mean (MMM), which is a metric used to
assess the bleaching potential of the sea surface temperature. MMM is
often calculated as the maximum monthly mean of the sea surface
temperature over a 30-year period, but here we are showing how to apply
the function with a smaller range of time.

``` r
sst_ts$max_monthly_ts |> 
  #Note you need to add group = 1 to plot correctly
  ggplot(aes(month, max_monthly_val, group = 1))+
  geom_line()+
  theme_bw()+
  labs(title = "MMM for Whitsunday Island",
       y = "Sea Surface Temperature (°C)")+
  theme(axis.title.x = element_blank())
```

![](07-API_Data_Access_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

Plotting time series if months are numbers instead of names.

``` r
sst_ts$max_monthly_ts |> 
  #Adding column with numbers
  mutate(mth = 1:12) |> 
  ggplot(aes(mth, max_monthly_val))+
  geom_line()+
  #Add labels with abbreviated month names
  scale_x_continuous(limits = c(1, 12), breaks = 1:12,
                     labels = month.abb)+
  theme_bw()+
  labs(title = "MMM for Whitsunday Island",
       y = "Sea Surface Temperature (°C)")+
  theme(axis.title.x = element_blank())
```

![](07-API_Data_Access_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

# Extra examples during workshop

Extra code created during training.

Extracting data for a single point from Whitsunday Island raster.

``` r
#Extracting data for a specific point
pt <- terra::extract(sst_gbr, vect("POINT (149 -20.2)"))

#Format data frame better for plotting
pt <- pt |> 
  #Pivot the table
  pivot_longer(!ID, names_to = "time", values_to = "sst") |> 
  #Adding time column - Get time from original raster
  mutate(time = time(sst_gbr),
         #Get time as year month
         year_month = format(time, "%Y-%m"))

#Calculating monthly means
pt_mean_sst <- pt |> 
  group_by(year_month) |>
  summarise(mean_sst = mean(sst, na.rm = T))

#Calculating maximum monthly values over entire period
pt_max_mth_sst <- pt_mean_sst |> 
  #Extracting month from year-month
  mutate(month = str_extract(year_month, "([0-9]{2})$",
                             group = 1)) |> 
  group_by(month) |> 
  summarise(monthly_max = max(mean_sst, na.rm = T))
  
#Calculating maximum SST by month over entire period
pt_max_mth_sst |> 
  ggplot(aes(x = month, y = monthly_max, group = 1))+
  geom_line()
```

![](07-API_Data_Access_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

``` r
#Plotting maximum monthly SST values
pt |> 
  group_by(year_month) |>
  summarise(max_sst_yr_mth = max(sst, na.rm = T)) |> 
  ggplot(aes(x = year_month, y = max_sst_yr_mth, group = 1))+
  geom_line()
```

![](07-API_Data_Access_files/figure-gfm/unnamed-chunk-10-2.png)<!-- -->
