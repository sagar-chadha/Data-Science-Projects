## NYC Taxi Trip Duration Prediction

![Screenshot](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/NYC-Taxi-Duration/NYC_Taxi_Trip_Duration_files/figure-markdown_github/NYC%20Taxi.jpeg)

The code here represents my final submission for the [competition](https://www.kaggle.com/c/nyc-taxi-trip-duration) by the same name hosted on Kaggle. I took up this project out of curiosity back when R was still the leading programming language for data science. It is one of my first attempts at predictive analytics - just an outlet for what I was learning about machine learning through an online course.

### The Task

The challenge was to build a model that predicts the total ride duration of taxi trips in New York City. The dataset was released by the NYC Taxi and Limousine Commission, and includes pickup time, geo-coordinates, number of passengers, and several other variables.

### Evaluation

The results were evaluated using the RMSLE (Root Mean Square Logarithmic Error). RMSLE is calculated using - 

![Screenshot](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/NYC-Taxi-Duration/NYC_Taxi_Trip_Duration_files/figure-markdown_github/formula.PNG)

As per this evaluation metric, I had an RMSLE score on the private leaderboard of 0.475 while the winning solution had a score of 0.289. Overall I stood 747th out of 1257 teams on the Public Leaderboard. Not bad for a first attempt. :)

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

### Results

This being my first experience with a prediction problem, I faced a lot of challenges on the way. The biggest learning for me was efficient manipulation of data and feature engineering. Going through the code posted by others on Kaggle showed me quite a few neat tricks that I managed to incorporate in my code as well. The two most important lessons from this project were - <br>
    1. Data cleaning and manipulation take up most of the time in tasks like these (I had only heard about this till then). <br>
    2. Comments are critical for understanding code. I learnt this after I had a hard time picking up where I had left off after even a few days gap.

### Repository Structure
* [`NYC_Taxi_Trip_Duration.Rmd`](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/NYC-Taxi-Duration/NYC_Taxi_Trip_Duration.Rmd) file is the R Notebook with the code I used to perform my analysis.
* [`NYC_Taxi_Trip_Duration.md`](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/NYC-Taxi-Duration/NYC_Taxi_Trip_Duration.md) is the neatly formatted markdown file with the complete analysis.
