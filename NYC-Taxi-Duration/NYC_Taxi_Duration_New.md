NYC Taxi Trip Duration Challenge
================================

The task at hand is to predict taxi trip durations in NYC using
variables such as pickup time, geo-coordinates, number of passengers,
etc.

Data Import
-----------

Let’s read in the training and testing data files. We will use the
stringsAsFactors FALSE since we don’t want all the strings to be
converted to factor variables as some of the date columns would be
stored as characters too.

``` r
test <- read.csv('test.csv', header = TRUE, stringsAsFactors = FALSE)
train <- read.csv('train.csv', header = TRUE, stringsAsFactors = FALSE)
```

What do the datasets look like?
-------------------------------

    ## [1] "The structure of the training data is shown below - "

    ## Observations: 1,458,644
    ## Variables: 11
    ## $ id                 <chr> "id2875421", "id2377394", "id3858529", "id3...
    ## $ vendor_id          <int> 2, 1, 2, 2, 2, 2, 1, 2, 1, 2, 2, 2, 2, 2, 2...
    ## $ pickup_datetime    <chr> "2016-03-14 17:24:55", "2016-06-12 00:43:35...
    ## $ dropoff_datetime   <chr> "2016-03-14 17:32:30", "2016-06-12 00:54:38...
    ## $ passenger_count    <int> 1, 1, 1, 1, 1, 6, 4, 1, 1, 1, 1, 4, 2, 1, 1...
    ## $ pickup_longitude   <dbl> -73.98215, -73.98042, -73.97903, -74.01004,...
    ## $ pickup_latitude    <dbl> 40.76794, 40.73856, 40.76394, 40.71997, 40....
    ## $ dropoff_longitude  <dbl> -73.96463, -73.99948, -74.00533, -74.01227,...
    ## $ dropoff_latitude   <dbl> 40.76560, 40.73115, 40.71009, 40.70672, 40....
    ## $ store_and_fwd_flag <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N"...
    ## $ trip_duration      <int> 455, 663, 2124, 429, 435, 443, 341, 1551, 2...

<br> <br>

    ## [1] "The structure of the test data is shown below - "

    ## Observations: 625,134
    ## Variables: 9
    ## $ id                 <chr> "id3004672", "id3505355", "id1217141", "id2...
    ## $ vendor_id          <int> 1, 1, 1, 2, 1, 1, 1, 1, 2, 2, 1, 2, 1, 2, 1...
    ## $ pickup_datetime    <chr> "2016-06-30 23:59:58", "2016-06-30 23:59:53...
    ## $ passenger_count    <int> 1, 1, 1, 1, 1, 1, 1, 2, 2, 1, 4, 1, 1, 1, 1...
    ## $ pickup_longitude   <dbl> -73.98813, -73.96420, -73.99744, -73.95607,...
    ## $ pickup_latitude    <dbl> 40.73203, 40.67999, 40.73758, 40.77190, 40....
    ## $ dropoff_longitude  <dbl> -73.99017, -73.95981, -73.98616, -73.98643,...
    ## $ dropoff_latitude   <dbl> 40.75668, 40.65540, 40.72952, 40.73047, 40....
    ## $ store_and_fwd_flag <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N"...

We see that we have a LOT of training and testing data - around
1,458,644 observations in the training data and 625,134 observations in
the test data. We note that in both the datasets, the datetime columns
`pickup_datetime` and `dropoff_datetime` have been imported as character
variables. These will need to be properly encoded. Two variables present
in the train data are not present in the test data - these are
`dropoff_datetime` and the output variable `trip_duration`. <br>

Let’s add these variables with the value 0 in the test data for
consistency.

``` r
test$dropoff_datetime <- 0 # add dropoff_datetime
test$trip_duration <- 0 # add trip_duration
```

Missing values
--------------

-   Missing values in the training data.

<!-- -->

    ##                 id          vendor_id    pickup_datetime 
    ##                  0                  0                  0 
    ##   dropoff_datetime    passenger_count   pickup_longitude 
    ##                  0                  0                  0 
    ##    pickup_latitude  dropoff_longitude   dropoff_latitude 
    ##                  0                  0                  0 
    ## store_and_fwd_flag      trip_duration 
    ##                  0                  0

<br>

-   Missing values in the test data.

<!-- -->

    ##                 id          vendor_id    pickup_datetime 
    ##                  0                  0                  0 
    ##    passenger_count   pickup_longitude    pickup_latitude 
    ##                  0                  0                  0 
    ##  dropoff_longitude   dropoff_latitude store_and_fwd_flag 
    ##                  0                  0                  0 
    ##   dropoff_datetime      trip_duration 
    ##                  0                  0

We find that these datasets do not have missing values. Good!

Data Exploration
----------------

Let’s look at a few rows of data and gather what we have.

    ##          id vendor_id     pickup_datetime    dropoff_datetime
    ## 1 id2875421         2 2016-03-14 17:24:55 2016-03-14 17:32:30
    ## 2 id2377394         1 2016-06-12 00:43:35 2016-06-12 00:54:38
    ## 3 id3858529         2 2016-01-19 11:35:24 2016-01-19 12:10:48
    ## 4 id3504673         2 2016-04-06 19:32:31 2016-04-06 19:39:40
    ## 5 id2181028         2 2016-03-26 13:30:55 2016-03-26 13:38:10
    ##   passenger_count pickup_longitude pickup_latitude dropoff_longitude
    ## 1               1        -73.98215        40.76794         -73.96463
    ## 2               1        -73.98042        40.73856         -73.99948
    ## 3               1        -73.97903        40.76394         -74.00533
    ## 4               1        -74.01004        40.71997         -74.01227
    ## 5               1        -73.97305        40.79321         -73.97292
    ##   dropoff_latitude store_and_fwd_flag trip_duration
    ## 1         40.76560                  N           455
    ## 2         40.73115                  N           663
    ## 3         40.71009                  N          2124
    ## 4         40.70672                  N           429
    ## 5         40.78252                  N           435

We see that we have the following variables - <br>

-   `id` - Trip ID. This wont be useful for the purpose of prediction.
-   `pickup_datetime`, `dropoff_datetime` - Time for pickup and dropoff
    of passengers.
-   `passenger_count` - count of passengers in the vehicle. Larger
    number of passengers may highlight a cab sharing arrangement and
    could be indicative of larger trip times.
-   `pickup_longitude`, `pickup_latitude` - location of the pickup.
-   `dropoff_longitude`, `dropoff_latitude` - location of the drop.
-   `trip_duration` - duration of the cab ride in seconds.

Let’s look at some of the variables in more detail - <br>

1.  `trip_duration`

``` r
summary(train$trip_duration)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##       1     397     662     959    1075 3526282

A summary above shows that while the 3rd quartile of `trip_duration` is
1075 seconds, we have a maximum of 3526282 seconds which is 980 hours!
This is impossible! The 99th percentile of the data is around 3440. We
will consider all the values above this to be outliers.

``` r
train <- train %>% filter(trip_duration <= quantile(train$trip_duration, 0.99))
```

We are left with 1444069 rows in the resulting dataset. These are still
enough datapoints for an efficient analysis. <br>

Next let’s convert to datetime the date/time variables that are
currently encoded as character.

``` r
train <- train %>% mutate(dropoff_datetime = ymd_hms(dropoff_datetime, tz = Sys.timezone()),
                          pickup_datetime = ymd_hms(pickup_datetime, tz = Sys.timezone()))
```

    ## Warning: 1 failed to parse.

    ## Warning: 1 failed to parse.

``` r
test <- test %>% mutate(pickup_datetime = ymd_hms(pickup_datetime, tz = Sys.timezone()))
```

Feature Engineering
-------------------

### Distance using latitude and longitude values.

We are given the latitude and longitude for the pickup and dropoff
locations. Let’s use these to calculate the distances between the pickup
and dropoff points. This will aid analysis as longer distances would
generally take a longer time.
