
## Exploring the Olympics dataset

Have you ever had a sports clash turn into a data analytics project? Well, this is what happened to me and a close group of friends during one casual conversation about sports!!

We decided to do some number crunching on 120 years of olympics to see which are the best performing countries at the olympics and what makes them great!

We got the [olympics dataset](https://www.kaggle.com/heesoo37/120-years-of-olympic-history-athletes-and-results) from kaggle, and decided to merge it with the country wise [gdp](https://www.kaggle.com/resulcaliskan/countries-gdps) and [population data](https://www.kaggle.com/centurion1986/countries-population).

With our weapons ready, its time to see who's BOSS!


```python
import pandas as pd
import numpy as np
%pylab inline
```

    Populating the interactive namespace from numpy and matplotlib
    

The first five rows of the olympics data are shown below. We have 271,116 rows and 15 columns. Variables include Name, Sex, Age, Height, Weight of the athlete, his team name, sport, event and the year, season, city of the olympics he/she took part in. In addition, the data captures the medal won (if any) by the athlete.


```python
olympics = pd.read_csv('C:/Users/sagar/Desktop/My Work/McCombs Stuff/MSBA Courses/Data Analytics Programming/Group Project/Data/athlete_events.csv')
print olympics.shape
print olympics.head()
```

    (271116, 15)
       ID                      Name Sex   Age  Height  Weight            Team  \
    0   1                 A Dijiang   M  24.0   180.0    80.0           China   
    1   2                  A Lamusi   M  23.0   170.0    60.0           China   
    2   3       Gunnar Nielsen Aaby   M  24.0     NaN     NaN         Denmark   
    3   4      Edgar Lindenau Aabye   M  34.0     NaN     NaN  Denmark/Sweden   
    4   5  Christine Jacoba Aaftink   F  21.0   185.0    82.0     Netherlands   
    
       NOC        Games  Year  Season       City          Sport  \
    0  CHN  1992 Summer  1992  Summer  Barcelona     Basketball   
    1  CHN  2012 Summer  2012  Summer     London           Judo   
    2  DEN  1920 Summer  1920  Summer  Antwerpen       Football   
    3  DEN  1900 Summer  1900  Summer      Paris     Tug-Of-War   
    4  NED  1988 Winter  1988  Winter    Calgary  Speed Skating   
    
                                  Event Medal  
    0       Basketball Men's Basketball   NaN  
    1      Judo Men's Extra-Lightweight   NaN  
    2           Football Men's Football   NaN  
    3       Tug-Of-War Men's Tug-Of-War  Gold  
    4  Speed Skating Women's 500 metres   NaN  
    

### Missing value check and Cleaning

#### 1) Missing Values
Printing column wise missing values we get-


```python
print olympics.isnull().sum()
```

    ID             0
    Name           0
    Sex            0
    Age         9474
    Height     60171
    Weight     62875
    Team           0
    NOC            0
    Games          0
    Year           0
    Season         0
    City           0
    Sport          0
    Event          0
    Medal     231333
    dtype: int64
    

We find that height, weight and Age have a lot of missing values. Medals have a NaN in about 2,31,333 rows. These can be explained since not all participating athletes would win medals. Let's replace these missing values by 'Did not win' or 'DNW'


```python
olympics['Medal'].fillna('DNW', inplace = True)
```

#### 2) NOC - National Olympic Committee. 
These are responsible for organizing their people's participation in the Olympics.
Are all NOCs linked to a unique team? We can find this out by taking a unique subset of just the NOC and team columns and taking a value count.


```python
print olympics.loc[:, ['NOC', 'Team']].drop_duplicates()['NOC'].value_counts().head()
```

    FRA    160
    USA     97
    GBR     96
    SWE     52
    NOR     46
    Name: NOC, dtype: int64
    

Hmm, This looks interesting. So NOC code 'FRA' is associated with 160 teams? That sounds prepostorous! Let's use a master of NOC to country mapping to correct this.

The NOC data has the NOC code and the corresponding Country Name. The first five rows of the data are shown below -


```python
# Lets read in the noc_country mapping first
noc_country = pd.read_csv('C:/Users/sagar/Desktop/My Work/McCombs Stuff/MSBA Courses/Data Analytics Programming/Group Project/Data/noc_regions.csv')
noc_country.drop('notes', axis = 1 , inplace = True)
noc_country.rename(columns = {'region':'Country'}, inplace = True)

print noc_country.head()
```

       NOC      Country
    0  AFG  Afghanistan
    1  AHO      Curacao
    2  ALB      Albania
    3  ALG      Algeria
    4  AND      Andorra
    

We now need to merge the original dataset with the NOC master using the **NOC code as the primary key**. This has to be a left join since we want all participating countries to remain in the data even if their NOC-Country is not found in the master. We can easily correct those manually.


```python
# merging
olympics_merge = olympics.merge(noc_country,
                                left_on = 'NOC',
                                right_on = 'NOC',
                                how = 'left')
```

Even after merging, we find that the below NOC codes in the Olympics data had no counterpart in the NOC master data.


```python
# Do we have NOCs that didnt have a matching country in the master?
print olympics_merge.loc[olympics_merge['Country'].isnull(),['NOC', 'Team']].drop_duplicates()
```

            NOC                      Team
    578     SGP                 Singapore
    6267    ROT  Refugee Olympic Athletes
    44376   SGP              June Climene
    61080   UNK                   Unknown
    64674   TUV                    Tuvalu
    80986   SGP                   Rika II
    108582  SGP               Singapore-2
    235895  SGP               Singapore-1
    

Looking at their 'Team' names we can manually insert the correct values into the olympics data.

Let's put these values in Country - <br>
    1. SGP - Singapore
    2. ROT - Refugee Olympic Athletes
    3. UNK - Unknown
    4. TUV - Tuvalu


```python
# Replace missing Teams by the values above.
#olympics_merge.loc[olympics_merge['Country'].isnull(), ['Country']] = olympics_merge['Team']

olympics_merge['Country'] = np.where(olympics_merge['NOC']=='SGP', 'Singapore', olympics_merge['Country'])
olympics_merge['Country'] = np.where(olympics_merge['NOC']=='ROT', 'Refugee Olympic Athletes', olympics_merge['Country'])
olympics_merge['Country'] = np.where(olympics_merge['NOC']=='UNK', 'Unknown', olympics_merge['Country'])
olympics_merge['Country'] = np.where(olympics_merge['NOC']=='TUV', 'Tuvalu', olympics_merge['Country'])


# Put these values from Country into Team
olympics_merge.drop('Team', axis = 1, inplace = True)
olympics_merge.rename(columns = {'Country': 'Team'}, inplace = True)
```

Checking again for mapping of NOC to team we find that each is mapped to a single value! Nice!

### Merge GDP data
To effectively study factors that affect the medal tally of a country, we need to import the Country-Year wise GDP data. THe GDP data has Country name, code, year and the GDP value. Some unnecessary columns _Indicator Name_ and _Indicator Code_ were removed. GDP data is shown below- 


```python
# Glance at the data.
w_gdp = pd.read_csv('C:/Users/sagar/Desktop/My Work/McCombs Stuff/MSBA Courses/Data Analytics Programming/Group Project\Data\world_gdp.csv', skiprows = 3)

# Remove unnecessary columns
w_gdp.drop(['Indicator Name', 'Indicator Code'], axis = 1, inplace = True)

# The columns are the years for which the GDP has been recorded. This needs to brought into a single column for efficient
# merging.
w_gdp = pd.melt(w_gdp, id_vars = ['Country Name', 'Country Code'], var_name = 'Year', value_name = 'GDP')

# convert the year column to numeric
w_gdp['Year'] = pd.to_numeric(w_gdp['Year'])

print w_gdp.head()
```

         Country Name Country Code  Year           GDP
    0     Afghanistan          AFG  1960  5.377778e+08
    1         Albania          ALB  1960           NaN
    2         Algeria          DZA  1960  2.723649e+09
    3  American Samoa          ASM  1960           NaN
    4         Andorra          AND  1960           NaN
    

Before we actually merge, lets check if NOCs in the olympics data match with those in the Country Code.


```python
print len(list(set(olympics_merge['NOC'].unique()) - set(w_gdp['Country Code'].unique())))
```

    108
    

So, 108 NOCs in the olympics dataset dont have representation in the gdp data country codes. Is the name of the country a better way to merge?


```python
print len(list(set(olympics_merge['Team'].unique()) - set(w_gdp['Country Name'].unique())))
```

    5
    

Aha! only 5! So maybe what I can do is, add a country code for each Team in the olympics dataset first and then merge using the Country Code. The data now has GDP data attached!


```python
# Merge to get country code
olympics_merge_ccode = olympics_merge.merge(w_gdp[['Country Name', 'Country Code']].drop_duplicates(),
                                            left_on = 'Team',
                                            right_on = 'Country Name',
                                            how = 'left')

olympics_merge_ccode.drop('Country Name', axis = 1, inplace = True)

# Merge to get gdp too
olympics_merge_gdp = olympics_merge_ccode.merge(w_gdp,
                                                left_on = ['Country Code', 'Year'],
                                                right_on = ['Country Code', 'Year'],
                                                how = 'left')

olympics_merge_gdp.drop('Country Name', axis = 1, inplace = True)
```

### Merge Population Data
Apart from GDP, population is also of import when looking at a countries performance. There are two train of thoughts for this - <br>
1. Higher populations mean a bigger talent pool to choose from,
2. Its not GDP alone but GDP per capita that decides how much resources a country has.

The first five rows of the population data are shown below. This dataset has Country, year and population statistic. Again, _Indicator Code_ and _Indicator Name_ were deemed unnecessary and removed.


```python
# Read in the population data
w_pop = pd.read_csv('C:/Users/sagar/Desktop/My Work/McCombs Stuff/MSBA Courses/Data Analytics Programming/Group Project\Data\world_pop.csv')

w_pop.drop(['Indicator Name', 'Indicator Code'], axis = 1, inplace = True)

w_pop = pd.melt(w_pop, id_vars = ['Country', 'Country Code'], var_name = 'Year', value_name = 'Population')

# Change the Year to integer type
w_pop['Year'] = pd.to_numeric(w_pop['Year'])

print w_pop.head()
```

           Country Country Code  Year  Population
    0        Aruba          ABW  1960     54211.0
    1  Afghanistan          AFG  1960   8996351.0
    2       Angola          AGO  1960   5643182.0
    3      Albania          ALB  1960   1608800.0
    4      Andorra          AND  1960     13411.0
    

Merging this data, we finally get a complete dataset with GDP and Population mapped. A few sample rows are shown below.


```python
# Merge olympics+GDP with population data
olympics_complete = olympics_merge_gdp.merge(w_pop,
                                            left_on = ['Country Code', 'Year'],
                                            right_on= ['Country Code', 'Year'],
                                            how = 'left')

olympics_complete.drop('Country', axis = 1, inplace = True)

print olympics_complete.head()
```

       ID                      Name Sex   Age  Height  Weight  NOC        Games  \
    0   1                 A Dijiang   M  24.0   180.0    80.0  CHN  1992 Summer   
    1   2                  A Lamusi   M  23.0   170.0    60.0  CHN  2012 Summer   
    2   3       Gunnar Nielsen Aaby   M  24.0     NaN     NaN  DEN  1920 Summer   
    3   4      Edgar Lindenau Aabye   M  34.0     NaN     NaN  DEN  1900 Summer   
    4   5  Christine Jacoba Aaftink   F  21.0   185.0    82.0  NED  1988 Winter   
    
       Year  Season       City          Sport                             Event  \
    0  1992  Summer  Barcelona     Basketball       Basketball Men's Basketball   
    1  2012  Summer     London           Judo      Judo Men's Extra-Lightweight   
    2  1920  Summer  Antwerpen       Football           Football Men's Football   
    3  1900  Summer      Paris     Tug-Of-War       Tug-Of-War Men's Tug-Of-War   
    4  1988  Winter    Calgary  Speed Skating  Speed Skating Women's 500 metres   
    
      Medal         Team Country Code           GDP    Population  
    0   DNW        China          CHN  4.269160e+11  1.164970e+09  
    1   DNW        China          CHN  8.560550e+12  1.350695e+09  
    2   DNW      Denmark          DNK           NaN           NaN  
    3  Gold      Denmark          DNK           NaN           NaN  
    4   DNW  Netherlands          NLD  2.585680e+11  1.476009e+07  
    

There are a lot of missing values in the resulting data - this is to be attributed to the countries not found in the GDP and population masters and also the fact that Population and GDP are only for 1961 onwards while Olympics data is from 1896. Therefore, let's consider only data from 1961 onwards.


```python
print olympics_complete.isnull().sum()
```

    ID                  0
    Name                0
    Sex                 0
    Age              9474
    Height          60171
    Weight          62875
    NOC                 0
    Games               0
    Year                0
    Season              0
    City                0
    Sport               0
    Event               0
    Medal               0
    Team                0
    Country Code     1245
    GDP             86777
    Population      64972
    dtype: int64
    


```python
# Lets take data from 1961 onwards only and for summer olympics only
olympics_complete_subset = olympics_complete.loc[(olympics_complete['Year'] > 1960) & (olympics_complete['Season'] == "Summer"), :]

# Reset row indices
olympics_complete_subset = olympics_complete_subset.reset_index()
```

### Exploratory Data Analysis

#### Who has the most medals across all editions of the olympics?
Medal tally is the sum of all medals won.

Let's create a column that captures whether or not a medal was won! It would be 1 if Medal column says Gold, Silver or Bronze and 0 otherwise.


```python
olympics_complete_subset['Medal_Won'] = np.where(olympics_complete_subset.loc[:,'Medal'] == 'DNW', 0, 1)
```

Before we find out the best olympics nations, we need to realise that each row of the data actually represents whether or not an athlete won a medal or not. This means that for team events, just a sum of the 'Medal_Won' column would give us an incorrect picture of the medal tally. 

We need to correct for this. First, lets analyse which events are team events. So, what events are team events?
#### If a team wins more than one gold medal for an event in an edition of the olympics, then that event is a team event.


```python
# Check whether number of medals won in a year for an event by a team exceeds 1. This indicates a team event.
identify_team_events = pd.pivot_table(olympics_complete_subset,
                                      index = ['Team', 'Year', 'Event'],
                                      columns = 'Medal',
                                      values = 'Medal_Won',
                                      aggfunc = 'sum',
                                     fill_value = 0).drop('DNW', axis = 1).reset_index()

identify_team_events = identify_team_events.loc[identify_team_events['Gold'] > 1, :]

team_sports = identify_team_events['Event'].unique()
```

The list below gives names of each event where in a single edition multiple golds were given. Going through the list, however, we found that some events have crept in which are not usually team events. Some examples include - <br>
    1. Gymnastics Women's Balance Beam
    2. Gymnastics Men's Horizontal Bar
    3. Swimming Women's 100 metres Freestyle
    4. Swimming Men's 50 metres Freestyle

Upon analysis, I found that these are actually single events but because two athletes had the same score/time, both were awarded the gold medal. We need to remove these events from the list of team sports


```python
remove_sports = ["Gymnastics Women's Balance Beam", "Gymnastics Men's Horizontal Bar", 
                 "Swimming Women's 100 metres Freestyle", "Swimming Men's 50 metres Freestyle"]

team_sports = list(set(team_sports) - set(remove_sports))
```

The next thing we need to do is add a column in the dataset that correctly identifies whether the event in the given record is a team event or a single event. We then use this column to correctly calculate the medal tally for each country.


```python
# if an event name matches with one in team sports, then it is a team event. Others are singles events.
team_event_mask = olympics_complete_subset['Event'].map(lambda x: x in team_sports)
single_event_mask = [not i for i in team_event_mask]

# rows where medal_won is 1
medal_mask = olympics_complete_subset['Medal_Won'] == 1

# Put 1 under team event if medal is won and event in team event list
olympics_complete_subset['Team_Event'] = np.where(team_event_mask & medal_mask, 1, 0)

# Put 1 under singles event if medal is won and event not in team event list
olympics_complete_subset['Single_Event'] = np.where(single_event_mask & medal_mask, 1, 0)

# Add an identifier for team/single event
olympics_complete_subset['Event_Category'] = olympics_complete_subset['Single_Event'] + \
olympics_complete_subset['Team_Event']
```

Now, its time to calculate medal tally agnostic of the team size - one gold means one gold for an event. To do this we divide the number of medals by the count of winning team members. How do we get the team members? Sum of event_category column should do that for us!


```python
medal_tally_agnostic = olympics_complete_subset.\
groupby(['Year', 'Team', 'Event', 'Medal'])[['Medal_Won', 'Event_Category']].\
agg('sum').reset_index()

medal_tally_agnostic['Medal_Won_Corrected'] = medal_tally_agnostic['Medal_Won']/medal_tally_agnostic['Event_Category']
```

Who are the greatest olympics playing nations of all time? With the corrected data, lets find out the total medal tally of the top 4 countries from 1964 to 2016!


```python
# Medal Tally.
medal_tally = medal_tally_agnostic.groupby(['Year','Team'])['Medal_Won_Corrected'].agg('sum').reset_index()

medal_tally_pivot = pd.pivot_table(medal_tally,
                     index = 'Team',
                     columns = 'Year',
                     values = 'Medal_Won_Corrected',
                     aggfunc = 'sum',
                     margins = True).sort_values('All', ascending = False)[1:5]

# print total medals won in the given period
print medal_tally_pivot.loc[:,'All']
```

    Team
    USA        1387.0
    Russia     1298.0
    Germany    1042.0
    China       545.0
    Name: All, dtype: float64
    

**USA, Russia, Germany** and **China** are the best countries of all time when it comes to medal tallies from 1964 to 2016. What do the yearwise medal tallies look like?


```python
# List of top countries
top_countries = ['USA', 'Russia', 'Germany', 'China']

year_team_medals = pd.pivot_table(medal_tally,
                                  index = 'Year',
                                  columns = 'Team',
                                  values = 'Medal_Won_Corrected',
                                  aggfunc = 'sum')[top_countries]

# plotting the medal tallies
year_team_medals.plot(linestyle = '-', marker = 'o', alpha = 0.9, figsize = (10,8), linewidth = 2)
xlabel('Olympic Year')
ylabel('Number of Medals')
title('Olympic Performance Comparison')
```




    Text(0.5,1,'Olympic Performance Comparison')




![png](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/ExploringTheOlympicsData/ExploringtheOlympicsData_files/ExploringtheOlympicsData_52_1.png)


**Interesting Insight 1**: The blank value at 1980 for USA and China is not a data error! In 1980, the United States led a boycott of the Summer Olympic Games in Moscow to protest the late 1979 Soviet invasion of Afghanistan. In total, 65 nations refused to participate in the games, whereas 80 countries sent athletes to compete, India being one of those.

**Interesting Insight 2**:The missing point at 1984 for Russia is no error either! The boycott of the 1984 Summer Olympics in Los Angeles followed four years after the U.S.-led boycott of the 1980 Summer Olympics in Moscow. The boycott involved 14 Eastern Bloc countries and allies, led by the Soviet Union, which initiated the boycott on May 8, 1984.

**Interesting Insight 3** - China did not win in the olympics till 1984 and it has shown excellent performance since then.

#### Lets plot a breakup of medal tally by the medal type - Gold, Silver, Bronze


```python
# List of top countries
top_countries = ['USA', 'Russia', 'Germany', 'China']

# row mask where countries match
row_mask_2 = medal_tally_agnostic['Team'].map(lambda x: x in top_countries)

# Pivot table to calculate sum of gold, silver and bronze medals for each country
medal_tally_specific = pd.pivot_table(medal_tally_agnostic[row_mask_2],
                                     index = ['Team'],
                                     columns = 'Medal',
                                     values = 'Medal_Won_Corrected',
                                     aggfunc = 'sum',
                                     fill_value = 0).drop('DNW', axis = 1)

# Re-order the columns so that they appear in order on the chart.
medal_tally_specific = medal_tally_specific.loc[:, ['Gold', 'Silver', 'Bronze']]

medal_tally_specific.plot(kind = 'bar', stacked = True, figsize = (8,6), rot = 0)
xlabel('Number of Medals')
ylabel('Country')
```




    Text(0,0.5,'Country')




![png](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/ExploringTheOlympicsData/ExploringtheOlympicsData_files/ExploringtheOlympicsData_55_1.png)


Surprisingly, countries are also in order of gold medal tallies!

### What sports are these countries best at? 

So, what we want to find here are the events for which a country has the most proportion of gold medals from its own gold medal tally across the years.


```python
# To get the sports, teams are best at, we now aggregate the medal_tally_agnostic dataframe as we did earlier.
best_team_sports = pd.pivot_table(medal_tally_agnostic[row_mask_2],
                                  index = ['Team', 'Event'],
                                  columns = 'Medal',
                                  values = 'Medal_Won_Corrected',
                                  aggfunc = 'sum',
                                  fill_value = 0).sort_values(['Team', 'Gold'], ascending = [True, False]).reset_index()

best_team_sports.drop(['Bronze', 'Silver', 'DNW'], axis = 1, inplace = True)
best_team_sports.columns = ['Team', 'Event', 'Gold_Medal_Count']

print best_team_sports.groupby('Team').head(5)
```

             Team                                          Event  Gold_Medal_Count
    0       China                     Diving Women's Springboard                 8
    1       China                   Table Tennis Women's Singles                 8
    2       China                        Diving Women's Platform                 7
    3       China                Weightlifting Men's Lightweight                 6
    4       China                      Badminton Women's Doubles                 5
    343   Germany             Equestrianism Mixed Dressage, Team                11
    344   Germany     Canoeing Women's Kayak Doubles, 500 metres                 7
    345   Germany                  Rowing Men's Quadruple Sculls                 7
    346   Germany                Rowing Women's Quadruple Sculls                 6
    347   Germany                   Athletics Men's Discus Throw                 5
    695    Russia         Wrestling Men's Heavyweight, Freestyle                 9
    696    Russia                      Fencing Men's Sabre, Team                 7
    697    Russia             Gymnastics Women's Team All-Around                 7
    698    Russia         Rhythmic Gymnastics Women's Individual                 7
    699    Russia   Wrestling Men's Light-Heavyweight, Freestyle                 7
    1049      USA     Swimming Men's 4 x 100 metres Medley Relay                13
    1050      USA  Swimming Men's 4 x 200 metres Freestyle Relay                11
    1051      USA           Athletics Men's 4 x 400 metres Relay                10
    1052      USA                     Athletics Men's 400 metres                10
    1053      USA                    Basketball Men's Basketball                10
    

We see that Chinese are good at diving - both men and women, Germans excel at equestrian sports, Russians likes to wrestle and Americans likes to swim.

## Size of Olympic contingent that these countries send to the Olympics!

How do the contingents for the various countries look like?


```python
# take for each year, the team, name of the athlete and gender of the athlete and drop duplicates. These are values
# where the same athlete is taking part in more than one sport.

# get rows with top countries
row_mask_3 = olympics_complete_subset['Team'].map(lambda x: x in top_countries)

year_team_gender = olympics_complete_subset.loc[row_mask_3, ['Year','Team', 'Name', 'Sex']].drop_duplicates()
```


```python
# Create a pivot table to count gender wise representation of each team in each year
year_team_gender_count = pd.pivot_table(year_team_gender,
                                        index = ['Year', 'Team'],
                                        columns = 'Sex',
                                        aggfunc = 'count').reset_index()

# rename columns as per column names in the 0th level
year_team_gender_count.columns = year_team_gender_count.columns.get_level_values(0)

# rename the columns appropriately
year_team_gender_count.columns = ['Year', 'Team', 'Female_Athletes', 'Male_Athletes']

# get total athletes per team-year
year_team_gender_count['Total_Athletes'] = year_team_gender_count['Female_Athletes'] + \
year_team_gender_count['Male_Athletes']
```


```python
# Separate country wise data

chi_data = year_team_gender_count[year_team_gender_count['Team'] == "China"]
chi_data.fillna(0, inplace = True)
chi_data.set_index('Year', inplace = True)

ger_data = year_team_gender_count[year_team_gender_count['Team'] == "Germany"]
ger_data.set_index('Year', inplace = True)

rus_data = year_team_gender_count[year_team_gender_count['Team'] == "Russia"]
rus_data.set_index('Year', inplace = True)

usa_data = year_team_gender_count[year_team_gender_count['Team'] == "USA"]
usa_data.set_index('Year', inplace = True)
```

    C:\Users\sagar\Anaconda2\lib\site-packages\pandas\core\frame.py:3787: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      downcast=downcast, **kwargs)
    

Plotting the contingent size with male and female athletes respectively for each country.


```python
# Plot the values of male, female and total athletes using bar charts and the line charts.
fig, ((ax1, ax2), (ax3, ax4)) = subplots(nrows = 2, ncols = 2, figsize = (20, 12), sharey = True)
fig.subplots_adjust(hspace = 0.3)

# Plot team Australia's contingent size
ax1.bar(chi_data.index.values, chi_data['Male_Athletes'], width = -1, align = 'edge', label = 'Male Athletes')
ax1.bar(chi_data.index.values, chi_data['Female_Athletes'], width = 1, align = 'edge', label = 'Female Athletes')
ax1.plot(chi_data.index.values, chi_data['Total_Athletes'], linestyle = ':', color = 'black', label = 'Total Athletes',
        marker = 'o')
ax1.set_title('Team China:\nComposition over the years')
ax1.set_ylabel('Number of Athletes')
ax1.legend(loc = 'best')

# Plot team USA's contingent size
ax2.bar(usa_data.index.values, usa_data['Male_Athletes'], width = -1, align = 'edge', label = 'Male Athletes')
ax2.bar(usa_data.index.values, usa_data['Female_Athletes'], width = 1, align = 'edge', label = 'Female Athletes')
ax2.plot(usa_data.index.values, usa_data['Total_Athletes'], linestyle = ':', color = 'black', label = 'Total Athletes',
        marker = 'o')
ax2.set_title('Team USA:\nComposition over the years')
ax2.set_ylabel('Number of Athletes')
ax2.legend(loc = 'best')

# Plot team Germany's contingent size
ax3.bar(ger_data.index.values, ger_data['Male_Athletes'], width = -1, align = 'edge', label = 'Male Athletes')
ax3.bar(ger_data.index.values, ger_data['Female_Athletes'], width = 1, align = 'edge', label = 'Female Athletes')
ax3.plot(ger_data.index.values, ger_data['Total_Athletes'], linestyle = ':', color = 'black', label = 'Total Athletes', 
         marker = 'o')
ax3.set_title('Team Germany:\nComposition over the years')
ax3.set_ylabel('Number of Athletes')
ax3.legend(loc = 'best')

# Plot team Russia's contingent size
ax4.bar(rus_data.index.values, rus_data['Male_Athletes'], width = -1, align = 'edge', label = 'Male Athletes')
ax4.bar(rus_data.index.values, rus_data['Female_Athletes'], width = 1, align = 'edge', label = 'Female Athletes')
ax4.plot(rus_data.index.values, rus_data['Total_Athletes'], linestyle = ':', color = 'black', label = 'Total Athletes',
        marker = 'o')
ax4.set_title('Team Russia:\nComposition over the years')
ax4.set_ylabel('Number of Athletes')
ax4.legend(loc = 'best')

show()
```


![png](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/ExploringTheOlympicsData/ExploringtheOlympicsData_files/ExploringtheOlympicsData_66_0.png)


### Does the size of the contingent relate with the number of medals?


```python
# Get year wise team wise athletes.
year_team_athelete = olympics_complete_subset.loc[row_mask_3, ['Year','Team', 'Name']].drop_duplicates()

# sum these up to get total contingent size.
contingent_size = pd.pivot_table(year_team_athelete,
                                 index = 'Year',
                                 columns = 'Team',
                                 values = 'Name',
                                 aggfunc = 'count')

fig, ((ax1, ax2), (ax3, ax4)) = subplots(nrows = 2,
                                         ncols = 2,
                                        figsize = (20,12))

fig.subplots_adjust(hspace = 0.3)

# Plot australia's medal tally and contingent size
contingent_size['China'].plot(ax = ax1, linestyle = '-', marker = 'o', linewidth = 2, color = 'red', 
                                  label = 'Contingent Size')
year_team_medals['China'].plot(ax = ax1, linestyle = '-', marker = 'o', linewidth = 2, color = 'black',
                                  label = 'Medal Tally')
ax1.plot(2008, contingent_size.loc[2008, 'China'], marker = '^', color = 'red', ms = 14)
ax1.plot(2008, year_team_medals.loc[2008, 'China'], marker = '^', color = 'black', ms = 14)
ax1.set_xlabel('Olympic Year')
ax1.set_ylabel('Number of Athletes/Medal Tally')
ax1.set_title('Team China\nContingent Size vs Medal Tally')
ax1.legend(loc = 'best')

# Plot USA's medal tally and contingent size
contingent_size['USA'].plot(ax = ax2, linestyle = '-', marker = 'o', linewidth = 2, color = 'blue',
                           label = 'Contingent Size')
year_team_medals['USA'].plot(ax = ax2, linestyle = '-', marker = 'o', linewidth = 2, color = 'black',
                            label = 'Medal Tally')
ax2.plot(1984, contingent_size.loc[1984, 'USA'], marker = '^', color = 'blue', ms = 14)
ax2.plot(1984, year_team_medals.loc[1984, 'USA'], marker = '^', color = 'black', ms = 14)
ax2.set_xlabel('Olympic Year')
ax2.set_ylabel('Number of Athletes/Medal Tally')
ax2.set_title('Team USA\nContingent Size vs Medal Tally')
ax2.legend(loc = 'best')

# Plot Germany's medal tally and contingent size
contingent_size['Germany'].plot(ax = ax3, linestyle = '-', marker = 'o', linewidth = 2, color = 'green',
                               label = 'Contingent Size')
year_team_medals['Germany'].plot(ax = ax3, linestyle = '-', marker = 'o', linewidth = 2, color = 'black',
                                label = 'Medal Tally')
ax3.plot(1972, year_team_medals.loc[1972, 'Germany'], marker = '^', color = 'black', ms = 14)
ax3.plot(1972, contingent_size.loc[1972, 'Germany'], marker = '^', color = 'green', ms = 14)
ax3.set_xlabel('Olympic Year')
ax3.set_ylabel('Number of Athletes/Medal Tally')
ax3.set_title('Team Germany\nContingent Size vs Medal Tally')
ax3.legend(loc = 'best')

# Plot Russia's medal tally and contingent size
contingent_size['Russia'].plot(ax = ax4, linestyle = '-', marker = 'o', linewidth = 2, color = 'orange',
                              label = 'Contingent Size')
year_team_medals['Russia'].plot(ax = ax4, linestyle = '-', marker = 'o', linewidth = 2, color = 'black',
                               label = 'Medal Tally')
ax4.plot(1980, contingent_size.loc[1980, 'Russia'], marker = '^', color = 'orange', ms = 14)
ax4.plot(1980, year_team_medals.loc[1980, 'Russia'], marker = '^', color = 'black', ms = 14)
ax4.set_xlabel('Olympic Year')
ax4.set_ylabel('Number of Athletes/Medal Tally')
ax4.set_title('Team Russia\nContingent Size vs Medal Tally')
ax4.legend(loc = 'best')

show()
```


![png](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/ExploringTheOlympicsData/ExploringtheOlympicsData_files/ExploringtheOlympicsData_68_0.png)



```python
# Lets merge contingent size and medals won!
year_team_medals_unstack = year_team_medals.unstack().reset_index()
year_team_medals_unstack.columns = ['Team','Year', 'Medal_Count']

contingent_size_unstack = contingent_size.unstack().reset_index()

contingent_size_unstack.columns = ['Team','Year', 'Contingent']

contingent_medals = contingent_size_unstack.merge(year_team_medals_unstack,
                                                 left_on = ['Team', 'Year'],
                                                 right_on = ['Team', 'Year'])

contingent_medals[['Contingent', 'Medal_Count']].corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Contingent</th>
      <th>Medal_Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Contingent</th>
      <td>1.000000</td>
      <td>0.703625</td>
    </tr>
    <tr>
      <th>Medal_Count</th>
      <td>0.703625</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



It is interesting to see that for each of these countries, a point of peak in the contingent size translates directly to a peak in the medal tally! These have been marked as large triangles on the plots. The same is confirmed by the correlation matrix above which shows a large positive correlation (0.703) between contingent size and medal tally.

**Interestingly, as we will see in the analysis below, the edition with peak contingents of all these nations is also the edition when the olympics was held in their home country**

## Are there some commonalities between the countries?
Are there common things they are all good at? The output shows the number of times a sport appears in the top 5 best sports of each country. Which sport has a number for multiple countries becomes a commonality


```python
team_commonalities = best_team_sports.merge(olympics_complete_subset.loc[:,['Sport', 'Event']].drop_duplicates(),
                                           left_on = 'Event',
                                           right_on = 'Event')

team_commonalities = team_commonalities.sort_values(['Team', 'Gold_Medal_Count'], ascending = [True, False])
team_commonalities = team_commonalities.groupby('Team').head(5).reset_index()

print pd.pivot_table(team_commonalities,
              index = 'Sport',
              columns = 'Team',
              values = 'Event',
              aggfunc = 'count',
              fill_value = 0,
              margins = True).sort_values('All', ascending = False)[1:]
```

    Team                 China  Germany  Russia  USA  All
    Sport                                                
    Athletics                0        1       0    2    3
    Diving                   2        0       0    0    2
    Rowing                   0        2       0    0    2
    Swimming                 0        0       0    2    2
    Wrestling                0        0       2    0    2
    Badminton                1        0       0    0    1
    Basketball               0        0       0    1    1
    Canoeing                 0        1       0    0    1
    Equestrianism            0        1       0    0    1
    Fencing                  0        0       1    0    1
    Gymnastics               0        0       1    0    1
    Rhythmic Gymnastics      0        0       1    0    1
    Table Tennis             1        0       0    0    1
    Weightlifting            1        0       0    0    1
    

#### Interesting Insight 3:
There is very little in common between these countries. It is perhaps why these are all at the top. Each competes and wins in its own area of expertise.

## Do teams enjoy a home advantage?
A home advantage is a phenomenon where a team does better when the olympics are being held in its own country than it fares in other countries. To keep the comparison fair, we would compare the performance of these teams in the year when the olympics was held at their home vs one edition earlier and one edition later.

#### Where was the olympics held in each of the years under consideration?


```python
print olympics_complete_subset[['Year', 'City']].drop_duplicates().sort_values('Year')
```

        Year            City
    74  1964           Tokyo
    11  1968     Mexico City
    12  1972          Munich
    37  1976        Montreal
    70  1980          Moskva
    10  1984     Los Angeles
    13  1988           Seoul
    0   1992       Barcelona
    3   1996         Atlanta
    2   2000          Sydney
    9   2004          Athina
    6   2008         Beijing
    1   2012          London
    7   2016  Rio de Janeiro
    

We see from the data that some cities have been captured incorrectly- <br>
1. Moskva should be 'Moscow'
2. Athina should be 'Athens'

Let's first correct these cities before we proceed with any analysis


```python
# Correct city names in the dataset
olympics_complete_subset['City'].replace(['Athina', 'Moskva'], ['Athens', 'Moscow'], inplace = True)
```

Now that the names have been corrected, let's map each of these cities to their countries.


```python
# city to country mapping dictionary
city_to_country = {'Tokyo': 'Japan',
                  'Mexico City': 'Mexico',
                  'Munich': 'Germany',
                  'Montreal': 'Canada',
                  'Moscow': 'Russia',
                  'Los Angeles': 'USA',
                  'Seoul': 'South Korea',
                  'Barcelona': 'Spain',
                  'Atlanta': 'USA',
                  'Sydney': 'Australia',
                  'Athens': 'Greece',
                  'Beijing': 'China',
                  'London': 'UK',
                  'Rio de Janeiro': 'Brazil'}

# Map cities to countries
olympics_complete_subset['Country_Host'] = olympics_complete_subset['City'].map(city_to_country)

#print the 
print olympics_complete_subset.loc[:, ['Year', 'Country_Host']].drop_duplicates().sort_values('Year')
```

        Year Country_Host
    74  1964        Japan
    11  1968       Mexico
    12  1972      Germany
    37  1976       Canada
    70  1980       Russia
    10  1984          USA
    13  1988  South Korea
    0   1992        Spain
    3   1996          USA
    2   2000    Australia
    9   2004       Greece
    6   2008        China
    1   2012           UK
    7   2016       Brazil
    

Phew! So somehow it has so happened that each of our top countries has atleast one instance of hosting in the considered time period! Fortuitous! :) 

One problem though, is that Russia hosted in a year when USA did _NOT_ participate. Since Russia and USA were the top contenders for medals in the period, it should be that Russia won a lot more than they would have in case USA participated. We would ideally need to control for this!

The approach for home advantage calculation would be to list host country against each year, playing nation and their total medal tally. Then we would compare performance one edition before and one edition after the olympics was held at that playing country's home.


```python
# Extract year, host nation and team name from the data
year_host_team = olympics_complete_subset[['Year', 'Country_Host', 'Team']].drop_duplicates()

# check rows where host country is the same as team
row_mask_4 = (year_host_team['Country_Host'] == year_host_team['Team'])

# add years in the year_host_team to capture one previous and one later year
year_host_team['Prev_Year'] = year_host_team['Year'] - 4
year_host_team['Next_Year'] = year_host_team['Year'] + 4

# Subset only where host nation and team were the same
year_host_team = year_host_team[row_mask_4]

# Calculate the medals won in each year where a team played at home. merge year_host_team with medal_tally on year and team
year_host_team_medal = year_host_team.merge(medal_tally,
                                           left_on = ['Year', 'Team'],
                                           right_on = ['Year', 'Team'],
                                           how = 'left')

year_host_team_medal.rename(columns = {'Medal_Won_Corrected' : 'Medal_Won_Host_Year'}, inplace = True)

# Calculate medals won by team in previous year
year_host_team_medal = year_host_team_medal.merge(medal_tally,
                                                 left_on = ['Prev_Year', 'Team'],
                                                 right_on = ['Year', 'Team'],
                                                 how = 'left')

year_host_team_medal.drop('Year_y', axis = 1, inplace = True)
year_host_team_medal.rename(columns = {'Medal_Won_Corrected': 'Medal_Won_Prev_Year',
                                      'Year_x':'Year'}, inplace = True)

# Calculate the medals won by the team the year after they hosted.
year_host_team_medal = year_host_team_medal.merge(medal_tally,
                                                 left_on = ['Next_Year', 'Team'],
                                                 right_on = ['Year', 'Team'],
                                                 how = 'left')

year_host_team_medal.drop('Year_y', axis = 1, inplace = True)
year_host_team_medal.rename(columns = {'Year_x': 'Year',
                                      'Medal_Won_Corrected' : 'Medal_Won_Next_Year'}, inplace = True)

# General formatting changes
year_host_team_medal.drop(['Prev_Year', 'Next_Year'], axis = 1, inplace = True)
year_host_team_medal.sort_values('Year', ascending = True, inplace = True)
year_host_team_medal.reset_index(inplace = True, drop = True)

# column re-ordering
year_host_team_medal = year_host_team_medal.loc[:, ['Year', 'Country_Host', 'Team', 'Medal_Won_Prev_Year', 'Medal_Won_Host_Year', 'Medal_Won_Next_Year']]

year_host_team_medal
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Country_Host</th>
      <th>Team</th>
      <th>Medal_Won_Prev_Year</th>
      <th>Medal_Won_Host_Year</th>
      <th>Medal_Won_Next_Year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1964</td>
      <td>Japan</td>
      <td>Japan</td>
      <td>NaN</td>
      <td>29.0</td>
      <td>25.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1968</td>
      <td>Mexico</td>
      <td>Mexico</td>
      <td>1.0</td>
      <td>9.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1972</td>
      <td>Germany</td>
      <td>Germany</td>
      <td>51.0</td>
      <td>106.0</td>
      <td>129.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1976</td>
      <td>Canada</td>
      <td>Canada</td>
      <td>5.0</td>
      <td>11.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1980</td>
      <td>Russia</td>
      <td>Russia</td>
      <td>125.0</td>
      <td>195.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1984</td>
      <td>USA</td>
      <td>USA</td>
      <td>NaN</td>
      <td>173.0</td>
      <td>94.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1988</td>
      <td>South Korea</td>
      <td>South Korea</td>
      <td>19.0</td>
      <td>33.0</td>
      <td>28.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1992</td>
      <td>Spain</td>
      <td>Spain</td>
      <td>4.0</td>
      <td>22.0</td>
      <td>17.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1996</td>
      <td>USA</td>
      <td>USA</td>
      <td>108.0</td>
      <td>101.0</td>
      <td>91.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2000</td>
      <td>Australia</td>
      <td>Australia</td>
      <td>41.0</td>
      <td>58.0</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2004</td>
      <td>Greece</td>
      <td>Greece</td>
      <td>13.0</td>
      <td>16.0</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2008</td>
      <td>China</td>
      <td>China</td>
      <td>64.0</td>
      <td>100.0</td>
      <td>89.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2012</td>
      <td>UK</td>
      <td>UK</td>
      <td>48.0</td>
      <td>65.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2016</td>
      <td>Brazil</td>
      <td>Brazil</td>
      <td>17.0</td>
      <td>19.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Excluding rows with Russia (1980) and USA (1984) because their biggest contender didn't participate in the olympics that time, we see that in **general team do better whilst playing at home!** 

## Does GDP play a part in medal tallies?


```python
import statsmodels.api as sm

year_team_gdp = olympics_complete_subset.loc[:, ['Year', 'Team', 'GDP']].drop_duplicates()

medal_tally_gdp = medal_tally.merge(year_team_gdp,
                                   left_on = ['Year', 'Team'],
                                   right_on = ['Year', 'Team'],
                                   how = 'left')

row_mask_5 = medal_tally_gdp['Medal_Won_Corrected'] > 0
row_mask_6 = medal_tally_gdp['Team'].map(lambda x: x in top_countries)

correlation = medal_tally_gdp.loc[row_mask_5, ['GDP', 'Medal_Won_Corrected']].corr()['Medal_Won_Corrected'][0]

plot(medal_tally_gdp.loc[row_mask_5, 'GDP'], 
     medal_tally_gdp.loc[row_mask_5, 'Medal_Won_Corrected'] , 
     linestyle = 'none', 
     marker = 'o',
    alpha = 0.4)
xlabel('Country GDP')

ylabel('Number of Medals')
title('GDP versus medal tally')
text(np.nanpercentile(medal_tally_gdp['GDP'], 99.6), 
     max(medal_tally_gdp['Medal_Won_Corrected']) - 50,
     "Correlation = " + str(correlation))
```




    Text(7.02006e+12,145,'Correlation = 0.622903299762')




![png](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/ExploringTheOlympicsData/ExploringtheOlympicsData_files/ExploringtheOlympicsData_85_1.png)


The plot shows a 0.622 correlation between GDP and medals won! Thats a significant correlation. **So GDP positively impacts the number of medals won by a team.**

## Who are the most accomplished athletes in each sport?

Lets define accomplished athletes as those with a high medal to participation ratio. This means the average number of medals they win with each participation. Participation would be counted as the number of events that they participated in.


```python
# lets create a data frame of athletes with the sport they participated in and the number of medals won.
ath_sport_medal = olympics_complete_subset.groupby(['Team','Name', 'Sport'])['Medal_Won'].agg('sum').reset_index()
ath_sport_medal.sort_values(['Sport', 'Medal_Won'], ascending = [True, False], inplace = True)

# keep only athletes who won medals
medal_mask = ath_sport_medal['Medal_Won'] > 0
ath_sport_medal = ath_sport_medal[medal_mask]

ath_sport_medal.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Team</th>
      <th>Name</th>
      <th>Sport</th>
      <th>Medal_Won</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>66110</th>
      <td>South Korea</td>
      <td>Kim Su-Nyeong</td>
      <td>Archery</td>
      <td>6</td>
    </tr>
    <tr>
      <th>65767</th>
      <td>South Korea</td>
      <td>Ki Bo-Bae</td>
      <td>Archery</td>
      <td>4</td>
    </tr>
    <tr>
      <th>66718</th>
      <td>South Korea</td>
      <td>Park Seong-Hyeon</td>
      <td>Archery</td>
      <td>4</td>
    </tr>
    <tr>
      <th>16642</th>
      <td>China</td>
      <td>Zhang Juanjuan</td>
      <td>Archery</td>
      <td>3</td>
    </tr>
    <tr>
      <th>39055</th>
      <td>Italy</td>
      <td>Marco Galiazzo</td>
      <td>Archery</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Now lets calculate the number of participations of each athlete. This will be sport wise.
ath_sport_appearance = olympics_complete_subset.groupby(['Team', 'Name', 'Sport'])['NOC'].agg('count').reset_index()

ath_sport_appearance.rename(columns = {'NOC': 'Event_Count'}, inplace = True)

ath_sport_appearance.head()

# lets merge these two.
ath_medal_appearance = ath_sport_medal.merge(ath_sport_appearance,
                                            left_on = ["Team", "Name", "Sport"],
                                            right_on = ['Team', 'Name', 'Sport'],
                                            how = "left")

# Calculate the medal per participation
ath_medal_appearance['Medal_Per_Participation'] = ath_medal_appearance['Medal_Won']/ath_medal_appearance['Event_Count']

ath_medal_appearance.sort_values(['Medal_Per_Participation', 'Medal_Won'], ascending = [False, False], inplace = True)

ath_medal_appearance.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Team</th>
      <th>Name</th>
      <th>Sport</th>
      <th>Medal_Won</th>
      <th>Event_Count</th>
      <th>Medal_Per_Participation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12179</th>
      <td>USA</td>
      <td>Natalie Anne Coughlin (-Hall)</td>
      <td>Swimming</td>
      <td>12</td>
      <td>12</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>150</th>
      <td>USA</td>
      <td>Frederick Carlton "Carl" Lewis</td>
      <td>Athletics</td>
      <td>10</td>
      <td>10</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4925</th>
      <td>Germany</td>
      <td>Isabelle Regina Werth</td>
      <td>Equestrianism</td>
      <td>10</td>
      <td>10</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>12184</th>
      <td>USA</td>
      <td>Gary Wayne Hall, Jr.</td>
      <td>Swimming</td>
      <td>10</td>
      <td>10</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>5296</th>
      <td>Italy</td>
      <td>Maria Valentina Vezzali</td>
      <td>Fencing</td>
      <td>9</td>
      <td>9</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>12191</th>
      <td>Germany</td>
      <td>Kornelia Ender (-Matthes, -Grummt)</td>
      <td>Swimming</td>
      <td>8</td>
      <td>8</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>12195</th>
      <td>USA</td>
      <td>Donald Arthur "Don" Schollander</td>
      <td>Swimming</td>
      <td>8</td>
      <td>8</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>12197</th>
      <td>USA</td>
      <td>Nathan Ghar-Jun Adrian</td>
      <td>Swimming</td>
      <td>8</td>
      <td>8</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4191</th>
      <td>UK</td>
      <td>Jason Francis Kenny</td>
      <td>Cycling</td>
      <td>7</td>
      <td>7</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4745</th>
      <td>China</td>
      <td>Wu Minxia</td>
      <td>Diving</td>
      <td>7</td>
      <td>7</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>



We see a lot of athletes with below 10 medals but a medal every time they participate. Let's filter only those that have more than 10 medals.


```python
# filter out athletes with less than 10 total medals.
ath_medal_appearance = ath_medal_appearance[ath_medal_appearance['Medal_Won'] >= 10]

ath_medal_appearance
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Team</th>
      <th>Name</th>
      <th>Sport</th>
      <th>Medal_Won</th>
      <th>Event_Count</th>
      <th>Medal_Per_Participation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12179</th>
      <td>USA</td>
      <td>Natalie Anne Coughlin (-Hall)</td>
      <td>Swimming</td>
      <td>12</td>
      <td>12</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>150</th>
      <td>USA</td>
      <td>Frederick Carlton "Carl" Lewis</td>
      <td>Athletics</td>
      <td>10</td>
      <td>10</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>4925</th>
      <td>Germany</td>
      <td>Isabelle Regina Werth</td>
      <td>Equestrianism</td>
      <td>10</td>
      <td>10</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>12184</th>
      <td>USA</td>
      <td>Gary Wayne Hall, Jr.</td>
      <td>Swimming</td>
      <td>10</td>
      <td>10</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>12176</th>
      <td>USA</td>
      <td>Michael Fred Phelps, II</td>
      <td>Swimming</td>
      <td>28</td>
      <td>30</td>
      <td>0.933333</td>
    </tr>
    <tr>
      <th>3614</th>
      <td>Germany</td>
      <td>Birgit Fischer-Schmidt</td>
      <td>Canoeing</td>
      <td>12</td>
      <td>13</td>
      <td>0.923077</td>
    </tr>
    <tr>
      <th>12177</th>
      <td>USA</td>
      <td>Dara Grace Torres (-Hoffman, -Minas)</td>
      <td>Swimming</td>
      <td>12</td>
      <td>13</td>
      <td>0.923077</td>
    </tr>
    <tr>
      <th>12181</th>
      <td>USA</td>
      <td>Mark Andrew Spitz</td>
      <td>Swimming</td>
      <td>11</td>
      <td>12</td>
      <td>0.916667</td>
    </tr>
    <tr>
      <th>12182</th>
      <td>USA</td>
      <td>Matthew Nicholas "Matt" Biondi</td>
      <td>Swimming</td>
      <td>11</td>
      <td>12</td>
      <td>0.916667</td>
    </tr>
    <tr>
      <th>12180</th>
      <td>USA</td>
      <td>Ryan Steven Lochte</td>
      <td>Swimming</td>
      <td>12</td>
      <td>14</td>
      <td>0.857143</td>
    </tr>
    <tr>
      <th>6809</th>
      <td>Czech Republic</td>
      <td>Vra slavsk (-Odloilov)</td>
      <td>Gymnastics</td>
      <td>10</td>
      <td>12</td>
      <td>0.833333</td>
    </tr>
    <tr>
      <th>12178</th>
      <td>USA</td>
      <td>Jennifer Elisabeth "Jenny" Thompson (-Cumpelik)</td>
      <td>Swimming</td>
      <td>12</td>
      <td>17</td>
      <td>0.705882</td>
    </tr>
    <tr>
      <th>6806</th>
      <td>Russia</td>
      <td>Nikolay Yefimovich Andrianov</td>
      <td>Gymnastics</td>
      <td>15</td>
      <td>24</td>
      <td>0.625000</td>
    </tr>
    <tr>
      <th>6810</th>
      <td>Japan</td>
      <td>Akinori Nakayama</td>
      <td>Gymnastics</td>
      <td>10</td>
      <td>16</td>
      <td>0.625000</td>
    </tr>
    <tr>
      <th>6811</th>
      <td>Russia</td>
      <td>Aleksandr Nikolayevich Dityatin</td>
      <td>Gymnastics</td>
      <td>10</td>
      <td>16</td>
      <td>0.625000</td>
    </tr>
    <tr>
      <th>6808</th>
      <td>Russia</td>
      <td>Aleksey Yuryevich Nemov</td>
      <td>Gymnastics</td>
      <td>12</td>
      <td>21</td>
      <td>0.571429</td>
    </tr>
    <tr>
      <th>6807</th>
      <td>Japan</td>
      <td>Sawao Kato</td>
      <td>Gymnastics</td>
      <td>12</td>
      <td>24</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>12183</th>
      <td>Germany</td>
      <td>Franziska van Almsick</td>
      <td>Swimming</td>
      <td>10</td>
      <td>23</td>
      <td>0.434783</td>
    </tr>
  </tbody>
</table>
</div>



We see that 9 out of 18 athletes in this list are from the United States of America and 8 of those are champion swimmers!

After the exploration, a lot of different questions come to mind. Can we predict the medal tally of a country? Is there some correlation between the age, height or weight of athletes in a sport? Lets try to answer these using machine learning techniques!

## Can we predict the medal tally of a country?

The exploration phase showed that the following factors affect a country's medal tally - 
1. GDP
2. Contingent Size
3. Population
4. Home field advantage


```python
# create the year, team contingent size
year_team_gender = olympics_complete_subset.loc[:,['Year','Team', 'Name', 'Sex']].drop_duplicates()

year_team_gender_count = pd.pivot_table(year_team_gender,
                                        index = ['Year', 'Team'],
                                        columns = 'Sex',
                                        aggfunc = 'count').reset_index()

# rename columns as per column names in the 0th level
year_team_gender_count.columns = year_team_gender_count.columns.get_level_values(0)

# rename the columns appropriately
year_team_gender_count.columns = ['Year', 'Team', 'Female_Athletes', 'Male_Athletes']
year_team_gender_count = year_team_gender_count.fillna(0)

# get total athletes per team-year
year_team_gender_count['Total_Athletes'] = year_team_gender_count['Female_Athletes'] + \
year_team_gender_count['Male_Athletes']

year_team_contingent = year_team_gender_count.loc[:, ['Year', 'Team','Total_Athletes']]
year_team_contingent.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Team</th>
      <th>Total_Athletes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1964</td>
      <td>Afghanistan</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1964</td>
      <td>Algeria</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1964</td>
      <td>Argentina</td>
      <td>102.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1964</td>
      <td>Australia</td>
      <td>243.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1964</td>
      <td>Austria</td>
      <td>56.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# get year, team and GDP - medal_tally_gdp has it
medal_tally_gdp.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Team</th>
      <th>Medal_Won_Corrected</th>
      <th>GDP</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1964</td>
      <td>Afghanistan</td>
      <td>0.0</td>
      <td>8.000000e+08</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1964</td>
      <td>Algeria</td>
      <td>0.0</td>
      <td>2.909352e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1964</td>
      <td>Argentina</td>
      <td>1.0</td>
      <td>2.560525e+10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1964</td>
      <td>Australia</td>
      <td>18.0</td>
      <td>2.378766e+10</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1964</td>
      <td>Austria</td>
      <td>0.0</td>
      <td>9.169984e+09</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Get year team population
year_team_pop = olympics_complete_subset.loc[:, ['Year', 'Team', 'Population']].drop_duplicates()
year_team_pop.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Team</th>
      <th>Population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1992</td>
      <td>China</td>
      <td>1.164970e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2012</td>
      <td>China</td>
      <td>1.350695e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2000</td>
      <td>Finland</td>
      <td>5.176209e+06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1996</td>
      <td>Finland</td>
      <td>5.124573e+06</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2008</td>
      <td>Norway</td>
      <td>4.768212e+06</td>
    </tr>
  </tbody>
</table>
</div>




```python
# merge all these datasets together
medal_gdp_population = medal_tally_gdp.merge(year_team_pop,
                                            left_on = ['Year', 'Team'],
                                            right_on = ['Year', 'Team'],
                                            how = 'left')

medal_gdp_pop_contingent = medal_gdp_population.merge(year_team_contingent,
                                                     left_on = ['Year', 'Team'],
                                                     right_on = ['Year', 'Team'],
                                                     how = 'left')
medal_gdp_pop_contingent.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Team</th>
      <th>Medal_Won_Corrected</th>
      <th>GDP</th>
      <th>Population</th>
      <th>Total_Athletes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1964</td>
      <td>Afghanistan</td>
      <td>0.0</td>
      <td>8.000000e+08</td>
      <td>9731361.0</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1964</td>
      <td>Algeria</td>
      <td>0.0</td>
      <td>2.909352e+09</td>
      <td>12295970.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1964</td>
      <td>Argentina</td>
      <td>1.0</td>
      <td>2.560525e+10</td>
      <td>21953929.0</td>
      <td>102.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1964</td>
      <td>Australia</td>
      <td>18.0</td>
      <td>2.378766e+10</td>
      <td>11167000.0</td>
      <td>243.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1964</td>
      <td>Austria</td>
      <td>0.0</td>
      <td>9.169984e+09</td>
      <td>7223801.0</td>
      <td>56.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# get host nation from the data
year_host = olympics_complete_subset.loc[:, ['Year', 'Country_Host']].drop_duplicates()

# merge this with the larger dataset
lin_model_data = medal_gdp_pop_contingent.merge(year_host,
                              left_on = 'Year',
                              right_on = 'Year',
                              how = 'left')

print lin_model_data.head()
```

       Year         Team  Medal_Won_Corrected           GDP  Population  \
    0  1964  Afghanistan                  0.0  8.000000e+08   9731361.0   
    1  1964      Algeria                  0.0  2.909352e+09  12295970.0   
    2  1964    Argentina                  1.0  2.560525e+10  21953929.0   
    3  1964    Australia                 18.0  2.378766e+10  11167000.0   
    4  1964      Austria                  0.0  9.169984e+09   7223801.0   
    
       Total_Athletes Country_Host  
    0             8.0        Japan  
    1             1.0        Japan  
    2           102.0        Japan  
    3           243.0        Japan  
    4            56.0        Japan  
    


```python
# Regression model
import statsmodels.api as sm
from patsy import dmatrices
from sklearn import metrics

# add indicator to see whether home advantage
lin_model_data['Home_adv'] = np.where(lin_model_data['Country_Host'] == lin_model_data['Team'], 1, 0)

# Add per capita GDP
lin_model_data['GDP_per_capita'] = lin_model_data['GDP']/lin_model_data['Population']

# remove NAs
lin_model_data.dropna(how = 'any', inplace = True)


# Lets see the distrbution of population and GDP in a histogram

fig, (ax1, ax2) = subplots(nrows = 1,
                          ncols = 2,
                          figsize = (14,5))

lin_model_data['GDP'].hist(bins = 15, ax = ax1)
ax1.set_title('GDP Distrbution')

lin_model_data['Population'].hist(bins = 15, ax = ax2)
ax2.set_title('Population Distribution')
```




    Text(0.5,1,'Population Distribution')




![png](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/ExploringTheOlympicsData/ExploringtheOlympicsData_files/ExploringtheOlympicsData_100_1.png)


The charts show that these variables are highly skewed. It would do better to do a logarithmic transform of these for better fit in the linear model.


```python
import math

# take the log transforms for the variables
lin_model_data['Log_Population'] = np.log(lin_model_data['Population'])
lin_model_data['Log_GDP'] = np.log(lin_model_data['GDP'])
```


```python
# regression model
y, X = dmatrices('Medal_Won_Corrected ~ Log_GDP + Log_Population + Total_Athletes + Home_adv + GDP_per_capita', 
                data = lin_model_data,
                return_type = 'dataframe')

model = sm.OLS(y, X)
result = model.fit()

print result.summary()
```

                                 OLS Regression Results                            
    ===============================================================================
    Dep. Variable:     Medal_Won_Corrected   R-squared:                       0.755
    Model:                             OLS   Adj. R-squared:                  0.754
    Method:                  Least Squares   F-statistic:                     1178.
    Date:                 Sun, 12 Aug 2018   Prob (F-statistic):               0.00
    Time:                         00:42:59   Log-Likelihood:                -6493.1
    No. Observations:                 1920   AIC:                         1.300e+04
    Df Residuals:                     1914   BIC:                         1.303e+04
    Df Model:                            5                                         
    Covariance Type:             nonrobust                                         
    ==================================================================================
                         coef    std err          t      P>|t|      [0.025      0.975]
    ----------------------------------------------------------------------------------
    Intercept         16.6034      2.202      7.540      0.000      12.285      20.922
    Log_GDP           -1.1224      0.160     -7.032      0.000      -1.435      -0.809
    Log_Population     0.3970      0.154      2.574      0.010       0.094       0.699
    Total_Athletes     0.1462      0.002     58.764      0.000       0.141       0.151
    Home_adv          -7.9719      2.173     -3.668      0.000     -12.234      -3.710
    GDP_per_capita  1.319e-05   1.48e-05      0.893      0.372   -1.58e-05    4.21e-05
    ==============================================================================
    Omnibus:                     1876.084   Durbin-Watson:                   2.074
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):           258647.618
    Skew:                           4.243   Prob(JB):                         0.00
    Kurtosis:                      59.224   Cond. No.                     2.63e+05
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 2.63e+05. This might indicate that there are
    strong multicollinearity or other numerical problems.
    

The result summary above shows an R^2 value of 0.75 which is good. This means that these variables account for 75% of all variation in the medal tally.


```python
# what is the error associated with this prediction?
y_predicted = result.predict(X)
print np.sqrt(metrics.mean_squared_error(y, y_predicted))
```

    7.11974243157
    

The above result shows that if these predictors are known, we can predict the medal tally of a country with an error of just ~7 medals! Thats impressive!

### Can we predict the sport an athlete plays using just height and weight?


```python
# How many unique sports are there?
print olympics_complete_subset['Sport'].unique()
```

    ['Basketball' 'Judo' 'Badminton' 'Sailing' 'Athletics' 'Handball'
     'Weightlifting' 'Wrestling' 'Water Polo' 'Hockey' 'Rowing' 'Swimming'
     'Football' 'Equestrianism' 'Boxing' 'Gymnastics' 'Taekwondo' 'Fencing'
     'Diving' 'Canoeing' 'Tennis' 'Shooting' 'Cycling' 'Softball' 'Archery'
     'Volleyball' 'Synchronized Swimming' 'Modern Pentathlon' 'Table Tennis'
     'Baseball' 'Rhythmic Gymnastics' 'Rugby Sevens' 'Trampolining'
     'Beach Volleyball' 'Triathlon' 'Golf']
    


```python
# extract sports data for males and height, weight statistics
base_data_male = olympics_complete_subset.loc[olympics_complete_subset['Sex'] == "M",
                                              ['Sport', 'Age','Weight', 'Height']].drop_duplicates()
```


```python
# Need to create y, X for the K nearest neighbors problem
Y, X = dmatrices('Sport ~ 0 + Weight + Height',
                data = base_data_male,
                return_type = 'dataframe')
```

Plotting for a few sports we see that, we can distinguish players who play athletics and weightlifting easily due to their unusual height or weight, but the rest of the sports look tricky to distinguish just on the basis of height and weight.


```python
fig, (ax1, ax2) = subplots(nrows = 1, ncols = 2, figsize = (13,6))

ax1.scatter(X['Weight'], X['Height'], c = Y['Sport[Weightlifting]'], cmap = 'bwr', marker = '.', alpha = 0.3)
ax1.set_title('Weightlifting vs Height and Weight')
ax1.set_xlabel('Weight')
ax1.set_ylabel('Height')

ax2.scatter(X['Weight'], X['Height'], c = Y['Sport[Athletics]'], cmap = 'bwr', marker = '.', alpha = 0.3)
ax2.set_title('Athletics vs Height and Weight')
ax2.set_xlabel('Weight')
ax2.set_ylabel('Height')
```




    Text(0,0.5,'Height')




![png](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/ExploringTheOlympicsData/ExploringtheOlympicsData_files/ExploringtheOlympicsData_112_1.png)


Lets see how well we can predict Weightlifting given height and weight.


```python
base_data_male['target'] = np.where(base_data_male['Sport'] == 'Weightlifting', 1, 0)
```


```python
# create the y and X matrices
from sklearn import model_selection
from sklearn import metrics
from sklearn import neighbors

Y, X = dmatrices('target ~ 0 + Weight + Height',
                data = base_data_male,
                return_type = 'dataframe')

y = Y['target'].values

accuracies = []

kfold = model_selection.StratifiedKFold(n_splits = 5, shuffle = True).split(X, y)

model = neighbors.KNeighborsClassifier(n_neighbors = 20,
                                      p = 2,
                                      weights = 'uniform')

for train, holdout in kfold:
    model.fit(X.iloc[train], y[train])
    prediction_on_test = model.predict(X.iloc[holdout])
    accuracies.append(metrics.accuracy_score(y[holdout], prediction_on_test))
```


```python
print np.mean(accuracies)
```

    0.969362274493
    

The above accuracies show that given a height and weight I can predict the sport as being or not being weightlifting with 96% accuracy! This was because the weightlifters were so clearly visibile as separate on the weight/height scatterplot.

Next, let's try predicting Athletics!


```python
# Predicting Athletics from height and weight
base_data_male['target'] = np.where(base_data_male['Sport'] == 'Athletics', 1, 0)

# create the y and X matrices
from sklearn import model_selection
from sklearn import metrics
from sklearn import neighbors

Y, X = dmatrices('target ~ 0 + Weight + Height',
                data = base_data_male,
                return_type = 'dataframe')

y = Y['target'].values

accuracies = []

kfold = model_selection.StratifiedKFold(n_splits = 5, shuffle = True).split(X, y)

model = neighbors.KNeighborsClassifier(n_neighbors = 20,
                                      p = 2,
                                      weights = 'uniform')

for train, holdout in kfold:
    model.fit(X.iloc[train], y[train])
    prediction_on_test = model.predict(X.iloc[holdout])
    accuracies.append(metrics.accuracy_score(y[holdout], prediction_on_test))
```


```python
print np.mean(accuracies)
```

    0.858976890238
    

Wow! We get 86% accurate predictions with k-nearest neighbors while predicting athletics.

## This was EXCITING! 

#### We learnt that -
1. USA, Russia, Germany and China are medal tally leaders in the olympics if we count beginning in 1964. They have all sent varying number of athletes into events over time and have fared better with higher contingent sizes. 
2. Women, who were not a very large part of the Olympics contingents earlier, now enjoy equal representation in the olympics teams. 
3. 1980 and 1984 were weird years, with USA not participating in 1980 due to Russia invading Afghanistan and then Russia retaliating in 1984 by not participating and starting the friendship games.
4. We found interesting insights by looking at the most successful sporting events for each of the top countries - 
5. While USA loves swimming, China likes diving, Russia has a penchant for wrestling and mostly all of them like athletics.
6. GDP impacts the number of medals won by a nation positively!
7. The most accomplished athletes list shows 50% Americans, mostly swimmers!
8. We can predict the medal tally of a country with an error (RMSE) of just ~7 medals using linear regression on - GDP, Population, GDP_per_capita and contingent size.
9. It is also possible to predict some sports that an athlete plays using just his height and weight. We used a very simple K-Nearest neighbors model. This can obviously be improved upon!

#### Hope you guys enjoyed it! Any comments are welcome! :)
