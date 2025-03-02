# NOAA_Degree_Heating_Week
Denisse Fierro Arcos
2023-10-18

- [Accessing and plotting NOAA’s Degree Heating Week
  data](#accessing-and-plotting-noaas-degree-heating-week-data)
- [Loading libraries](#loading-libraries)
- [Connecting to RIMReP collection via
  API](#connecting-to-rimrep-collection-via-api)

# Accessing and plotting NOAA’s Degree Heating Week data

This notebook will demonstrate how to access the RIMReP collection for
[NOAA’s Coral Reef Watch - Degree Heating Week
(DHW)](https://stac.staging.reefdata.io/browser/collections/noaa-crw/items/noaa-crw-dhw?.language=en&.asset=asset-data).
This dataset provides coral bleaching heat stress index derived from
satellite data a global scale with a temporal resolution of 1-day and a
horizontal spatial resolution of about 5 km ($0.05^{\circ}$). The
dataset available in the DMS is [version 3.1 of the NOAA’s Coral Reef
Watch - Degree Heating Week](https://coralreefwatch.noaa.gov/index.php),
which includes data from January 1, 1985 to present.

In this notebook, we will use the `connect_dms_dataset` function from
the `useful_functions.R` script, which allows us to access data from the
RIMReP DMS API. We also have the option of including spatial and
temporal boundaries to extract the data of interest.

# Loading libraries

``` r
#Loading useful_functions script
source("useful_functions.R")
#Mapping
library(terra)
```

# Connecting to RIMReP collection via API

From the STAC catalogue entry for the [DHW
dataset](https://stac.reefdata.io/browser/collections/noaa-crw/items/noaa-crw-dhw?.language=en&.asset=asset-data),
we can get the link to the API from the *Additional Resources* section
of the page (bottom left).

We will show how to get data from this API link for the period between
2023-01-01 and 2023-01-07. For this example, we will select data for
coastal waters up to 150 km away from the coastline between Daintree and
Cairns.

**Note:** Before running the code chunk below, ensure you head over to
our dashboard:
[https://dashboard.reefdata.io/](https://dashboard.staging.reefdata.io/).
Then click on the blue **Copy token to clipboard** button. After you run
the code chunk below, you will see a pop-up box requesting for this
token. Paste your token in the box and press *OK* to continue. If you do
not have this token, you will not be able to access our API, please
contact the RIMReP team to set up an account by emailing
<rimrep-dms@aims.gov.au>.

``` r
#Defining API URL (obtained from STAC catalogue)
base_url <- "https://pygeoapi.reefdata.io/collections/noaa-crw-chs-dhw"

#Defining variable of interest (obtained from STAC catalogue)
variable_name <- "degree_heating_week"

#Connecting to DMS to extract data
ras_dhw <- connect_dms_dataset(base_url, variable_name,
                           start_time = "2023-01-01", end_time = "2023-01-07", 
                           lon_limits = c(145.30, 146.90),
                           lat_limits = c(-17, -16.30))
```

    Warning: No 'access_token' and no user credentials were provided as input.

    Checking if 'CLIENT_ID' variable exists.

    Warning: No 'access_token' and user credentials were provided as input.

    Checking if 'CLIENT_SECRET' variable exists.

    Access token retrieved successfully.

This function will query the API and return the data in gridded form
(i.e., raster). Once the data is loaded to memory, we can plot it using
the `terra` library.

``` r
plot(ras_dhw)
```

![](06-NOAA_Degree_Heating_Week_files/figure-commonmark/unnamed-chunk-3-1.png)
