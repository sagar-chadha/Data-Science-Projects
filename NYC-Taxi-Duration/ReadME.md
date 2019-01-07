## NYC Taxi Duration Prediction

![Screenshot](https://github.com/sagar-chadha/NYC-Taxi-duration/blob/master/NYC%20Taxi.jpeg)

This code represents my submission code for the [competition](https://www.kaggle.com/c/nyc-taxi-trip-duration) by the same name hosted by Kaggle. Overall I stood 747th out of 1257 teams on the Public Leaderboard. It was one of my forays into the world of predictive analytics using R and I learnt a lot from it.

### The Problem

The challenge was to build a model that predicts the total ride duration of taxi trips in New York City. The dataset was released by the NYC Taxi and Limousine Commission, which includes pickup time, geo-coordinates, number of passengers, and several other variables.

### Evaluation

The results were evaluated using the RMSLE (Root Mean Square Logarithmic Error). RMSLE is calculated using - 

![Screenshot](https://github.com/sagar-chadha/NYC-Taxi-duration/blob/master/formula.PNG)

As per this evaluation metric, I had a RMSLE score on the private leaderboard of 0.475 while the winning solution had a score of 0.289.

### Solution Approach

I used some of the more popular R libraries for this analysis - `dplyr` and `lubridate` for data manipulation, `ggplot2` for visualization and `caret` for building predictive models. The approach was very straightforward - <br>

* **Check data for missing values** - I didn't find any in this dataset.
* **Data Anomalies** - <br>
    1. Values greater than 1000000 seconds in the trip duration column - rows removed.
    2. Trip duration very low for large trip distances - rows removed.
* **Feature Engineering** - Following features were added to the data <br>
    1. The distance between the pickup and dropoff locations.
    2. Day of pickup.
    3. Binary variable to identify whether pickup is on a weekend.
    4. Binary variable to identify rush hours on weekdays between 8 - 10am and 6-9pm.
    5. Binary Variable for major holidays.
    6. Categorical variable to identify time of the day - Early Morning, Morning, Afternoon and Night.
    7. Frequency of pickups and dropoffs to a neighbourhood.
    8. Wherever ride distance was 0, added a binary variable to capture ride cancellations.
* Log transforms of the distance and duration variables.

### Models Tried

I tried a very basic linear regression model and boosted regression models using the `gbm` library. The boosted models were optimised using the `caret` library. The best performing model was chosen for the final submission.

### What I Learnt

This being my first experience with a prediction problem, I faced a lot of challenges on the way. The biggest challenge for me was efficient manipulation of data and feature engineering. Going through others' attempts on Kaggle showed me quite a few neat tricks for doing this that I managed to incorporate in my code as well. In the end I realised that most of my code revolves only around data manipulation, cleaning, etc. forever solidifying in my mind that it is indeed data cleaning that takes up most of the time in such projects.
