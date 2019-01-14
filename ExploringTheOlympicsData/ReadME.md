## Exploring 120 years of Olympics Data

![image](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/ExploringTheOlympicsData/ExploringtheOlympicsData_files/1200px-Olympic_rings_without_rims.svg.png)

An innocent discussion around intra-mural sports with some friends at University turned into a grand debate about sporting prowess and we decided to turn it into a Python project!

### The task
We decided to work with the ['120 years of Olympic History'](https://www.kaggle.com/heesoo37/120-years-of-olympic-history-athletes-and-results) dataset on Kaggle and use that to see which are the best performing countries at the olympics and what makes them great! <br>

In addition, I wanted to use what I had learnt about predictive modelling and see if - 
* I can predict the medal tally of a nation based on their GDP and population. 
* K-Nearest Neighbors can help me predict the sport a person plays using just their weight and height.

### Concepts Used
* Dataframe manipulations using the Pandas library - renaming and dropping columns, using melt and pivot to reshape tables, groupby, etc.
* Various visualizations using the Matplotlib library - Stacked bar charts, line charts.
* Linear regression and K-Nearest Neighbors

### Results
One thing I learnt that I really didn't need data for is that I, being an Indian, should not have challenged an American when it comes to Olympics medal tallies (My bad!). But in all seriousness, I learnt a lot about the Olympics from this analysis, some of it being - 
* In 1980, the United States led a boycott of the Summer Olympic Games in Moscow to protest the late 1979 Soviet invasion of Afghanistan. In total, 65 nations refused to participate in the games, whereas 80 countries sent athletes to compete, India being one of those.
* The boycott of the 1984 Summer Olympics in Los Angeles followed four years after the U.S.-led boycott of the 1980 Summer Olympics in Moscow. The boycott involved 14 Eastern Bloc countries and allies, led by the Soviet Union, which initiated the boycott on May 8, 1984.
* Medal tallies for various sporting events show that the Chinese are good at diving - both men and women, Germans excel at equestrian sports, Russians likes to wrestle and Americans likes to swim.
* Each of the top 4 medal winners have very little in common in terms of sports that they excel at. It is perhaps why these are all at the top. Each competes and wins in its own area of expertise.
... and many more!

### Repository Structure
The `ExploringTheOlympicsData.ipynb` file has the code I used to perform my analysis. I used Jupyter notebook and Python 2.7.
