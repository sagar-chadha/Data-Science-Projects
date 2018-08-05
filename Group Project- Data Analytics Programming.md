
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
    


```python
# Read in the data set
olympics = pd.read_csv('./Data/athlete_events.csv')
print olympics.head()
```

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
    

### Data exploration and Basic Hygiene

#### 1) Missing Values


```python
olympics.isnull().sum()
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



We find that height, weight and Age have a lot of missing values. Medals have a NaN in about 2,31,333 rows. These can be explained since not all participating athletes would win medals.

Let's replace these missing values by 'Did not win' or 'DNW'


```python
olympics['Medal'].fillna('DNW', inplace = True)
```


```python
# As expected the NaNs in the 'Medal' column disappear!
olympics.isnull().sum()
```




    ID            0
    Name          0
    Sex           0
    Age        9474
    Height    60171
    Weight    62875
    Team          0
    NOC           0
    Games         0
    Year          0
    Season        0
    City          0
    Sport         0
    Event         0
    Medal         0
    dtype: int64



#### 2) NOC - National Olympic Committee. 
These are responsible for organizing their people's participation in the Olympics.
Are all NOCs linked to a unique team? We can find this out by taking a unique subset of just the NOC and team columns and taking a value count.


```python
olympics.loc[:, ['NOC', 'Team']].drop_duplicates()['NOC'].value_counts().head()
```




    FRA    160
    USA     97
    GBR     96
    SWE     52
    NOR     46
    Name: NOC, dtype: int64



Hmm, This looks interesting. So NOC code 'FRA' is associated with 160 teams? That sounds prepostorous! Let's do a groupby and verify this.


```python
olympics_NOC_Team = olympics.groupby(['NOC', 'Team'])[['Medal']].agg('count').reset_index()

len(olympics_NOC_Team.loc[olympics_NOC_Team['NOC'] == 'FRA', :])
```




    160



So this is true! Okay let's use a master of NOC to country mapping to correct this.


```python
# Lets read in the noc_country mapping first
noc_country = pd.read_csv('./Data/noc_regions.csv')
noc_country.drop('notes', axis = 1 , inplace = True)
noc_country.rename(columns = {'region':'Country'}, inplace = True)

noc_country.head()
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
      <th>NOC</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AFG</td>
      <td>Afghanistan</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AHO</td>
      <td>Curacao</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ALB</td>
      <td>Albania</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ALG</td>
      <td>Algeria</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AND</td>
      <td>Andorra</td>
    </tr>
  </tbody>
</table>
</div>



We now need to merge the original dataset with the NOC master using the NOC code as the primary key. This has to be a left join since we want all participating countries to remain in the data even if their NOC-Country is not found in the master. We can easily correct those manually.


```python
# merging
olympics_merge = olympics.merge(noc_country,
                                left_on = 'NOC',
                                right_on = 'NOC',
                                how = 'left')

olympics_merge.head()
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
      <th>ID</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>Height</th>
      <th>Weight</th>
      <th>Team</th>
      <th>NOC</th>
      <th>Games</th>
      <th>Year</th>
      <th>Season</th>
      <th>City</th>
      <th>Sport</th>
      <th>Event</th>
      <th>Medal</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>A Dijiang</td>
      <td>M</td>
      <td>24.0</td>
      <td>180.0</td>
      <td>80.0</td>
      <td>China</td>
      <td>CHN</td>
      <td>1992 Summer</td>
      <td>1992</td>
      <td>Summer</td>
      <td>Barcelona</td>
      <td>Basketball</td>
      <td>Basketball Men's Basketball</td>
      <td>DNW</td>
      <td>China</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>A Lamusi</td>
      <td>M</td>
      <td>23.0</td>
      <td>170.0</td>
      <td>60.0</td>
      <td>China</td>
      <td>CHN</td>
      <td>2012 Summer</td>
      <td>2012</td>
      <td>Summer</td>
      <td>London</td>
      <td>Judo</td>
      <td>Judo Men's Extra-Lightweight</td>
      <td>DNW</td>
      <td>China</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Gunnar Nielsen Aaby</td>
      <td>M</td>
      <td>24.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Denmark</td>
      <td>DEN</td>
      <td>1920 Summer</td>
      <td>1920</td>
      <td>Summer</td>
      <td>Antwerpen</td>
      <td>Football</td>
      <td>Football Men's Football</td>
      <td>DNW</td>
      <td>Denmark</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Edgar Lindenau Aabye</td>
      <td>M</td>
      <td>34.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Denmark/Sweden</td>
      <td>DEN</td>
      <td>1900 Summer</td>
      <td>1900</td>
      <td>Summer</td>
      <td>Paris</td>
      <td>Tug-Of-War</td>
      <td>Tug-Of-War Men's Tug-Of-War</td>
      <td>Gold</td>
      <td>Denmark</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Christine Jacoba Aaftink</td>
      <td>F</td>
      <td>21.0</td>
      <td>185.0</td>
      <td>82.0</td>
      <td>Netherlands</td>
      <td>NED</td>
      <td>1988 Winter</td>
      <td>1988</td>
      <td>Winter</td>
      <td>Calgary</td>
      <td>Speed Skating</td>
      <td>Speed Skating Women's 500 metres</td>
      <td>DNW</td>
      <td>Netherlands</td>
    </tr>
  </tbody>
</table>
</div>



Do we having NOCs in olympics that are not found in the NOC master data?


```python
# Do we have NOCs that didnt have a matching country in the master?
olympics_merge.loc[olympics_merge['Country'].isnull(),['NOC', 'Team']].drop_duplicates()
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
      <th>NOC</th>
      <th>Team</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>578</th>
      <td>SGP</td>
      <td>Singapore</td>
    </tr>
    <tr>
      <th>6267</th>
      <td>ROT</td>
      <td>Refugee Olympic Athletes</td>
    </tr>
    <tr>
      <th>44376</th>
      <td>SGP</td>
      <td>June Climene</td>
    </tr>
    <tr>
      <th>61080</th>
      <td>UNK</td>
      <td>Unknown</td>
    </tr>
    <tr>
      <th>64674</th>
      <td>TUV</td>
      <td>Tuvalu</td>
    </tr>
    <tr>
      <th>80986</th>
      <td>SGP</td>
      <td>Rika II</td>
    </tr>
    <tr>
      <th>108582</th>
      <td>SGP</td>
      <td>Singapore-2</td>
    </tr>
    <tr>
      <th>235895</th>
      <td>SGP</td>
      <td>Singapore-1</td>
    </tr>
  </tbody>
</table>
</div>



So, we see that SGP, ROT, UNK and TUV from the olympics data find no match in the NOC master data. Looking at their 'Team' names we can manually insert the correct values into the olympics data.

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


```python
olympics_merge.loc[:, ['NOC', 'Team']].drop_duplicates()['NOC'].value_counts().head()
```




    PNG    1
    BUL    1
    UGA    1
    TKM    1
    SCG    1
    Name: NOC, dtype: int64



### Merge GDP data


```python
# Glance at the data.
w_gdp = pd.read_csv('.\Data\world_gdp.csv', skiprows = 3)
w_gdp.head()
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
      <th>Country Name</th>
      <th>Country Code</th>
      <th>Indicator Name</th>
      <th>Indicator Code</th>
      <th>1960</th>
      <th>1961</th>
      <th>1962</th>
      <th>1963</th>
      <th>1964</th>
      <th>1965</th>
      <th>...</th>
      <th>2007</th>
      <th>2008</th>
      <th>2009</th>
      <th>2010</th>
      <th>2011</th>
      <th>2012</th>
      <th>2013</th>
      <th>2014</th>
      <th>2015</th>
      <th>2016</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>GDP (current US$)</td>
      <td>NY.GDP.MKTP.CD</td>
      <td>5.377778e+08</td>
      <td>5.488889e+08</td>
      <td>5.466667e+08</td>
      <td>7.511112e+08</td>
      <td>8.000000e+08</td>
      <td>1.006667e+09</td>
      <td>...</td>
      <td>9.843842e+09</td>
      <td>1.019053e+10</td>
      <td>1.248694e+10</td>
      <td>1.593680e+10</td>
      <td>1.793024e+10</td>
      <td>2.053654e+10</td>
      <td>2.004633e+10</td>
      <td>2.005019e+10</td>
      <td>1.921556e+10</td>
      <td>1.946902e+10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>ALB</td>
      <td>GDP (current US$)</td>
      <td>NY.GDP.MKTP.CD</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>1.070101e+10</td>
      <td>1.288135e+10</td>
      <td>1.204421e+10</td>
      <td>1.192695e+10</td>
      <td>1.289087e+10</td>
      <td>1.231978e+10</td>
      <td>1.277628e+10</td>
      <td>1.322824e+10</td>
      <td>1.133526e+10</td>
      <td>1.186387e+10</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>GDP (current US$)</td>
      <td>NY.GDP.MKTP.CD</td>
      <td>2.723649e+09</td>
      <td>2.434777e+09</td>
      <td>2.001469e+09</td>
      <td>2.703015e+09</td>
      <td>2.909352e+09</td>
      <td>3.136259e+09</td>
      <td>...</td>
      <td>1.349770e+11</td>
      <td>1.710010e+11</td>
      <td>1.372110e+11</td>
      <td>1.612070e+11</td>
      <td>2.000190e+11</td>
      <td>2.090590e+11</td>
      <td>2.097550e+11</td>
      <td>2.138100e+11</td>
      <td>1.658740e+11</td>
      <td>1.590490e+11</td>
    </tr>
    <tr>
      <th>3</th>
      <td>American Samoa</td>
      <td>ASM</td>
      <td>GDP (current US$)</td>
      <td>NY.GDP.MKTP.CD</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>5.200000e+08</td>
      <td>5.630000e+08</td>
      <td>6.780000e+08</td>
      <td>5.760000e+08</td>
      <td>5.740000e+08</td>
      <td>6.440000e+08</td>
      <td>6.410000e+08</td>
      <td>6.430000e+08</td>
      <td>6.590000e+08</td>
      <td>6.580000e+08</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Andorra</td>
      <td>AND</td>
      <td>GDP (current US$)</td>
      <td>NY.GDP.MKTP.CD</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>4.016972e+09</td>
      <td>4.007353e+09</td>
      <td>3.660531e+09</td>
      <td>3.355695e+09</td>
      <td>3.442063e+09</td>
      <td>3.164615e+09</td>
      <td>3.281585e+09</td>
      <td>3.350736e+09</td>
      <td>2.811489e+09</td>
      <td>2.858518e+09</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 61 columns</p>
</div>



Looking at the data we find that 'Indicator Name' and 'Indicator Code' have only one value in the entire column. We can therefore safely remove these columns from the dataset.


```python
w_gdp.drop(['Indicator Name', 'Indicator Code'], axis = 1, inplace = True)
```


```python
w_gdp.head()
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
      <th>Country Name</th>
      <th>Country Code</th>
      <th>1960</th>
      <th>1961</th>
      <th>1962</th>
      <th>1963</th>
      <th>1964</th>
      <th>1965</th>
      <th>1966</th>
      <th>1967</th>
      <th>...</th>
      <th>2007</th>
      <th>2008</th>
      <th>2009</th>
      <th>2010</th>
      <th>2011</th>
      <th>2012</th>
      <th>2013</th>
      <th>2014</th>
      <th>2015</th>
      <th>2016</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>5.377778e+08</td>
      <td>5.488889e+08</td>
      <td>5.466667e+08</td>
      <td>7.511112e+08</td>
      <td>8.000000e+08</td>
      <td>1.006667e+09</td>
      <td>1.400000e+09</td>
      <td>1.673333e+09</td>
      <td>...</td>
      <td>9.843842e+09</td>
      <td>1.019053e+10</td>
      <td>1.248694e+10</td>
      <td>1.593680e+10</td>
      <td>1.793024e+10</td>
      <td>2.053654e+10</td>
      <td>2.004633e+10</td>
      <td>2.005019e+10</td>
      <td>1.921556e+10</td>
      <td>1.946902e+10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>ALB</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>1.070101e+10</td>
      <td>1.288135e+10</td>
      <td>1.204421e+10</td>
      <td>1.192695e+10</td>
      <td>1.289087e+10</td>
      <td>1.231978e+10</td>
      <td>1.277628e+10</td>
      <td>1.322824e+10</td>
      <td>1.133526e+10</td>
      <td>1.186387e+10</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2.723649e+09</td>
      <td>2.434777e+09</td>
      <td>2.001469e+09</td>
      <td>2.703015e+09</td>
      <td>2.909352e+09</td>
      <td>3.136259e+09</td>
      <td>3.039835e+09</td>
      <td>3.370843e+09</td>
      <td>...</td>
      <td>1.349770e+11</td>
      <td>1.710010e+11</td>
      <td>1.372110e+11</td>
      <td>1.612070e+11</td>
      <td>2.000190e+11</td>
      <td>2.090590e+11</td>
      <td>2.097550e+11</td>
      <td>2.138100e+11</td>
      <td>1.658740e+11</td>
      <td>1.590490e+11</td>
    </tr>
    <tr>
      <th>3</th>
      <td>American Samoa</td>
      <td>ASM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>5.200000e+08</td>
      <td>5.630000e+08</td>
      <td>6.780000e+08</td>
      <td>5.760000e+08</td>
      <td>5.740000e+08</td>
      <td>6.440000e+08</td>
      <td>6.410000e+08</td>
      <td>6.430000e+08</td>
      <td>6.590000e+08</td>
      <td>6.580000e+08</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Andorra</td>
      <td>AND</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>4.016972e+09</td>
      <td>4.007353e+09</td>
      <td>3.660531e+09</td>
      <td>3.355695e+09</td>
      <td>3.442063e+09</td>
      <td>3.164615e+09</td>
      <td>3.281585e+09</td>
      <td>3.350736e+09</td>
      <td>2.811489e+09</td>
      <td>2.858518e+09</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 59 columns</p>
</div>




```python
# The columns are the years for which the GDP has been recorded. This needs to brought into a single column for efficient
# merging.
w_gdp = pd.melt(w_gdp, id_vars = ['Country Name', 'Country Code'], var_name = 'Year', value_name = 'GDP')

# convert the year column to numeric
w_gdp['Year'] = pd.to_numeric(w_gdp['Year'])

w_gdp.head()
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
      <th>Country Name</th>
      <th>Country Code</th>
      <th>Year</th>
      <th>GDP</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>1960</td>
      <td>5.377778e+08</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>ALB</td>
      <td>1960</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>1960</td>
      <td>2.723649e+09</td>
    </tr>
    <tr>
      <th>3</th>
      <td>American Samoa</td>
      <td>ASM</td>
      <td>1960</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Andorra</td>
      <td>AND</td>
      <td>1960</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Before we actually merge, lets check if NOCs in the olympics data match with those in the Country Code.


```python
len(list(set(olympics_merge['NOC'].unique()) - set(w_gdp['Country Code'].unique())))
```




    108



So, 108 NOCs in the olympics dataset dont have representation in the gdp data country codes. Is the name of the country a better way to merge?


```python
len(list(set(olympics_merge['Team'].unique()) - set(w_gdp['Country Name'].unique())))
```




    5



Aha! only 5! What countries are these? So maybe what I can do is, add a country code for each Team in the olympics dataset to help ease things.


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


```python
olympics_merge_gdp.head()
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
      <th>ID</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>Height</th>
      <th>Weight</th>
      <th>NOC</th>
      <th>Games</th>
      <th>Year</th>
      <th>Season</th>
      <th>City</th>
      <th>Sport</th>
      <th>Event</th>
      <th>Medal</th>
      <th>Team</th>
      <th>Country Code</th>
      <th>GDP</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>A Dijiang</td>
      <td>M</td>
      <td>24.0</td>
      <td>180.0</td>
      <td>80.0</td>
      <td>CHN</td>
      <td>1992 Summer</td>
      <td>1992</td>
      <td>Summer</td>
      <td>Barcelona</td>
      <td>Basketball</td>
      <td>Basketball Men's Basketball</td>
      <td>DNW</td>
      <td>China</td>
      <td>CHN</td>
      <td>4.269160e+11</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>A Lamusi</td>
      <td>M</td>
      <td>23.0</td>
      <td>170.0</td>
      <td>60.0</td>
      <td>CHN</td>
      <td>2012 Summer</td>
      <td>2012</td>
      <td>Summer</td>
      <td>London</td>
      <td>Judo</td>
      <td>Judo Men's Extra-Lightweight</td>
      <td>DNW</td>
      <td>China</td>
      <td>CHN</td>
      <td>8.560550e+12</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Gunnar Nielsen Aaby</td>
      <td>M</td>
      <td>24.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>DEN</td>
      <td>1920 Summer</td>
      <td>1920</td>
      <td>Summer</td>
      <td>Antwerpen</td>
      <td>Football</td>
      <td>Football Men's Football</td>
      <td>DNW</td>
      <td>Denmark</td>
      <td>DNK</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Edgar Lindenau Aabye</td>
      <td>M</td>
      <td>34.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>DEN</td>
      <td>1900 Summer</td>
      <td>1900</td>
      <td>Summer</td>
      <td>Paris</td>
      <td>Tug-Of-War</td>
      <td>Tug-Of-War Men's Tug-Of-War</td>
      <td>Gold</td>
      <td>Denmark</td>
      <td>DNK</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Christine Jacoba Aaftink</td>
      <td>F</td>
      <td>21.0</td>
      <td>185.0</td>
      <td>82.0</td>
      <td>NED</td>
      <td>1988 Winter</td>
      <td>1988</td>
      <td>Winter</td>
      <td>Calgary</td>
      <td>Speed Skating</td>
      <td>Speed Skating Women's 500 metres</td>
      <td>DNW</td>
      <td>Netherlands</td>
      <td>NLD</td>
      <td>2.585680e+11</td>
    </tr>
  </tbody>
</table>
</div>



### Merge Population Data


```python
# Read in the population data
w_pop = pd.read_csv('.\Data\world_pop.csv')
w_pop.head()
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
      <th>Country</th>
      <th>Country Code</th>
      <th>Indicator Name</th>
      <th>Indicator Code</th>
      <th>1960</th>
      <th>1961</th>
      <th>1962</th>
      <th>1963</th>
      <th>1964</th>
      <th>1965</th>
      <th>...</th>
      <th>2007</th>
      <th>2008</th>
      <th>2009</th>
      <th>2010</th>
      <th>2011</th>
      <th>2012</th>
      <th>2013</th>
      <th>2014</th>
      <th>2015</th>
      <th>2016</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Aruba</td>
      <td>ABW</td>
      <td>Population, total</td>
      <td>SP.POP.TOTL</td>
      <td>54211.0</td>
      <td>55438.0</td>
      <td>56225.0</td>
      <td>56695.0</td>
      <td>57032.0</td>
      <td>57360.0</td>
      <td>...</td>
      <td>101220.0</td>
      <td>101353.0</td>
      <td>101453.0</td>
      <td>101669.0</td>
      <td>102053.0</td>
      <td>102577.0</td>
      <td>103187.0</td>
      <td>103795.0</td>
      <td>104341.0</td>
      <td>104822</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>Population, total</td>
      <td>SP.POP.TOTL</td>
      <td>8996351.0</td>
      <td>9166764.0</td>
      <td>9345868.0</td>
      <td>9533954.0</td>
      <td>9731361.0</td>
      <td>9938414.0</td>
      <td>...</td>
      <td>26616792.0</td>
      <td>27294031.0</td>
      <td>28004331.0</td>
      <td>28803167.0</td>
      <td>29708599.0</td>
      <td>30696958.0</td>
      <td>31731688.0</td>
      <td>32758020.0</td>
      <td>33736494.0</td>
      <td>34656032</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Angola</td>
      <td>AGO</td>
      <td>Population, total</td>
      <td>SP.POP.TOTL</td>
      <td>5643182.0</td>
      <td>5753024.0</td>
      <td>5866061.0</td>
      <td>5980417.0</td>
      <td>6093321.0</td>
      <td>6203299.0</td>
      <td>...</td>
      <td>20997687.0</td>
      <td>21759420.0</td>
      <td>22549547.0</td>
      <td>23369131.0</td>
      <td>24218565.0</td>
      <td>25096150.0</td>
      <td>25998340.0</td>
      <td>26920466.0</td>
      <td>27859305.0</td>
      <td>28813463</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Albania</td>
      <td>ALB</td>
      <td>Population, total</td>
      <td>SP.POP.TOTL</td>
      <td>1608800.0</td>
      <td>1659800.0</td>
      <td>1711319.0</td>
      <td>1762621.0</td>
      <td>1814135.0</td>
      <td>1864791.0</td>
      <td>...</td>
      <td>2970017.0</td>
      <td>2947314.0</td>
      <td>2927519.0</td>
      <td>2913021.0</td>
      <td>2905195.0</td>
      <td>2900401.0</td>
      <td>2895092.0</td>
      <td>2889104.0</td>
      <td>2880703.0</td>
      <td>2876101</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Andorra</td>
      <td>AND</td>
      <td>Population, total</td>
      <td>SP.POP.TOTL</td>
      <td>13411.0</td>
      <td>14375.0</td>
      <td>15370.0</td>
      <td>16412.0</td>
      <td>17469.0</td>
      <td>18549.0</td>
      <td>...</td>
      <td>82683.0</td>
      <td>83861.0</td>
      <td>84462.0</td>
      <td>84449.0</td>
      <td>83751.0</td>
      <td>82431.0</td>
      <td>80788.0</td>
      <td>79223.0</td>
      <td>78014.0</td>
      <td>77281</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 61 columns</p>
</div>



Indicator Name and Indicator Code have the same value for all rows. We can safely remove these rows.


```python
w_pop.drop(['Indicator Name', 'Indicator Code'], axis = 1, inplace = True)
```

Reshape the data to bring years into a single column.


```python
w_pop = pd.melt(w_pop, id_vars = ['Country', 'Country Code'], var_name = 'Year', value_name = 'Population')

# Change the Year to integer type
w_pop['Year'] = pd.to_numeric(w_pop['Year'])
w_pop.head()
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
      <th>Country</th>
      <th>Country Code</th>
      <th>Year</th>
      <th>Population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Aruba</td>
      <td>ABW</td>
      <td>1960</td>
      <td>54211.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>1960</td>
      <td>8996351.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Angola</td>
      <td>AGO</td>
      <td>1960</td>
      <td>5643182.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Albania</td>
      <td>ALB</td>
      <td>1960</td>
      <td>1608800.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Andorra</td>
      <td>AND</td>
      <td>1960</td>
      <td>13411.0</td>
    </tr>
  </tbody>
</table>
</div>



Before we merge, lets check the matches between Country code in olympics_merge_gdp and country code in w_pop.


```python
list(set(olympics_merge_gdp['Country Code']) - set(w_pop['Country Code'])) 
```




    [nan, 'SSD', 'ERI']




```python
olympics_complete = olympics_merge_gdp.merge(w_pop,
                                            left_on = ['Country Code', 'Year'],
                                            right_on= ['Country Code', 'Year'],
                                            how = 'left')

olympics_complete.drop('Country', axis = 1, inplace = True)
```


```python
olympics_complete.head()
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
      <th>ID</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>Height</th>
      <th>Weight</th>
      <th>NOC</th>
      <th>Games</th>
      <th>Year</th>
      <th>Season</th>
      <th>City</th>
      <th>Sport</th>
      <th>Event</th>
      <th>Medal</th>
      <th>Team</th>
      <th>Country Code</th>
      <th>GDP</th>
      <th>Population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>A Dijiang</td>
      <td>M</td>
      <td>24.0</td>
      <td>180.0</td>
      <td>80.0</td>
      <td>CHN</td>
      <td>1992 Summer</td>
      <td>1992</td>
      <td>Summer</td>
      <td>Barcelona</td>
      <td>Basketball</td>
      <td>Basketball Men's Basketball</td>
      <td>DNW</td>
      <td>China</td>
      <td>CHN</td>
      <td>4.269160e+11</td>
      <td>1.164970e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>A Lamusi</td>
      <td>M</td>
      <td>23.0</td>
      <td>170.0</td>
      <td>60.0</td>
      <td>CHN</td>
      <td>2012 Summer</td>
      <td>2012</td>
      <td>Summer</td>
      <td>London</td>
      <td>Judo</td>
      <td>Judo Men's Extra-Lightweight</td>
      <td>DNW</td>
      <td>China</td>
      <td>CHN</td>
      <td>8.560550e+12</td>
      <td>1.350695e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Gunnar Nielsen Aaby</td>
      <td>M</td>
      <td>24.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>DEN</td>
      <td>1920 Summer</td>
      <td>1920</td>
      <td>Summer</td>
      <td>Antwerpen</td>
      <td>Football</td>
      <td>Football Men's Football</td>
      <td>DNW</td>
      <td>Denmark</td>
      <td>DNK</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Edgar Lindenau Aabye</td>
      <td>M</td>
      <td>34.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>DEN</td>
      <td>1900 Summer</td>
      <td>1900</td>
      <td>Summer</td>
      <td>Paris</td>
      <td>Tug-Of-War</td>
      <td>Tug-Of-War Men's Tug-Of-War</td>
      <td>Gold</td>
      <td>Denmark</td>
      <td>DNK</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Christine Jacoba Aaftink</td>
      <td>F</td>
      <td>21.0</td>
      <td>185.0</td>
      <td>82.0</td>
      <td>NED</td>
      <td>1988 Winter</td>
      <td>1988</td>
      <td>Winter</td>
      <td>Calgary</td>
      <td>Speed Skating</td>
      <td>Speed Skating Women's 500 metres</td>
      <td>DNW</td>
      <td>Netherlands</td>
      <td>NLD</td>
      <td>2.585680e+11</td>
      <td>1.476009e+07</td>
    </tr>
  </tbody>
</table>
</div>



There are a lot of missing values in the data, this is to be attributed to the countries not found in the GDP and population masters and also the fact that Population and GDP are only for 1961 onwards while Olympics data is from 1896.


```python
olympics_complete.isnull().sum()
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
```


```python
olympics_complete_subset = olympics_complete_subset.reset_index()
```


```python
olympics_complete_subset.head()
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
      <th>index</th>
      <th>ID</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>Height</th>
      <th>Weight</th>
      <th>NOC</th>
      <th>Games</th>
      <th>Year</th>
      <th>Season</th>
      <th>City</th>
      <th>Sport</th>
      <th>Event</th>
      <th>Medal</th>
      <th>Team</th>
      <th>Country Code</th>
      <th>GDP</th>
      <th>Population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1</td>
      <td>A Dijiang</td>
      <td>M</td>
      <td>24.0</td>
      <td>180.0</td>
      <td>80.0</td>
      <td>CHN</td>
      <td>1992 Summer</td>
      <td>1992</td>
      <td>Summer</td>
      <td>Barcelona</td>
      <td>Basketball</td>
      <td>Basketball Men's Basketball</td>
      <td>DNW</td>
      <td>China</td>
      <td>CHN</td>
      <td>4.269160e+11</td>
      <td>1.164970e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>2</td>
      <td>A Lamusi</td>
      <td>M</td>
      <td>23.0</td>
      <td>170.0</td>
      <td>60.0</td>
      <td>CHN</td>
      <td>2012 Summer</td>
      <td>2012</td>
      <td>Summer</td>
      <td>London</td>
      <td>Judo</td>
      <td>Judo Men's Extra-Lightweight</td>
      <td>DNW</td>
      <td>China</td>
      <td>CHN</td>
      <td>8.560550e+12</td>
      <td>1.350695e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>31</td>
      <td>12</td>
      <td>Jyri Tapani Aalto</td>
      <td>M</td>
      <td>31.0</td>
      <td>172.0</td>
      <td>70.0</td>
      <td>FIN</td>
      <td>2000 Summer</td>
      <td>2000</td>
      <td>Summer</td>
      <td>Sydney</td>
      <td>Badminton</td>
      <td>Badminton Men's Singles</td>
      <td>DNW</td>
      <td>Finland</td>
      <td>FIN</td>
      <td>1.255400e+11</td>
      <td>5.176209e+06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>32</td>
      <td>13</td>
      <td>Minna Maarit Aalto</td>
      <td>F</td>
      <td>30.0</td>
      <td>159.0</td>
      <td>55.5</td>
      <td>FIN</td>
      <td>1996 Summer</td>
      <td>1996</td>
      <td>Summer</td>
      <td>Atlanta</td>
      <td>Sailing</td>
      <td>Sailing Women's Windsurfer</td>
      <td>DNW</td>
      <td>Finland</td>
      <td>FIN</td>
      <td>1.320990e+11</td>
      <td>5.124573e+06</td>
    </tr>
    <tr>
      <th>4</th>
      <td>33</td>
      <td>13</td>
      <td>Minna Maarit Aalto</td>
      <td>F</td>
      <td>34.0</td>
      <td>159.0</td>
      <td>55.5</td>
      <td>FIN</td>
      <td>2000 Summer</td>
      <td>2000</td>
      <td>Summer</td>
      <td>Sydney</td>
      <td>Sailing</td>
      <td>Sailing Women's Windsurfer</td>
      <td>DNW</td>
      <td>Finland</td>
      <td>FIN</td>
      <td>1.255400e+11</td>
      <td>5.176209e+06</td>
    </tr>
  </tbody>
</table>
</div>



### Data Visualization

### Who has the most medals across all editions of the olympics?
Medal tally is the sum of all medals won.

Let's create a column that captures whether or not a medal was won! It would be 1 if Medal column says Gold, Silver or Bronze and 0 otherwise.


```python
olympics_complete_subset['Medal_Won'] = np.where(olympics_complete_subset.loc[:,'Medal'] == 'DNW', 0, 1)
```

Who are the greatest olympics playing nations of all time? Lets make a pivot table to find out!


```python
# Medal Tally.
medal_tally = olympics_complete_subset.groupby(['Year','Team'])['Medal_Won'].agg('sum').reset_index()
#medal_tally['Year'] = pd.to_datetime(medal_tally['Year'], format = "%Y")
```


```python
pd.pivot_table(medal_tally,
              index = 'Team',
              columns = 'Year',
              values = 'Medal_Won',
              aggfunc = 'sum',
              margins = True).sort_values('All', ascending = False)[1:5]
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
      <th>Year</th>
      <th>1964</th>
      <th>1968</th>
      <th>1972</th>
      <th>1976</th>
      <th>1980</th>
      <th>1984</th>
      <th>1988</th>
      <th>1992</th>
      <th>1996</th>
      <th>2000</th>
      <th>2004</th>
      <th>2008</th>
      <th>2012</th>
      <th>2016</th>
      <th>All</th>
    </tr>
    <tr>
      <th>Team</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>USA</th>
      <td>169.0</td>
      <td>166.0</td>
      <td>171.0</td>
      <td>164.0</td>
      <td>NaN</td>
      <td>352.0</td>
      <td>207.0</td>
      <td>224.0</td>
      <td>259.0</td>
      <td>242.0</td>
      <td>263.0</td>
      <td>317.0</td>
      <td>248.0</td>
      <td>264.0</td>
      <td>3046</td>
    </tr>
    <tr>
      <th>Russia</th>
      <td>174.0</td>
      <td>192.0</td>
      <td>214.0</td>
      <td>286.0</td>
      <td>442.0</td>
      <td>NaN</td>
      <td>300.0</td>
      <td>220.0</td>
      <td>115.0</td>
      <td>187.0</td>
      <td>189.0</td>
      <td>142.0</td>
      <td>140.0</td>
      <td>115.0</td>
      <td>2716</td>
    </tr>
    <tr>
      <th>Germany</th>
      <td>116.0</td>
      <td>103.0</td>
      <td>253.0</td>
      <td>273.0</td>
      <td>264.0</td>
      <td>158.0</td>
      <td>296.0</td>
      <td>198.0</td>
      <td>124.0</td>
      <td>118.0</td>
      <td>149.0</td>
      <td>99.0</td>
      <td>94.0</td>
      <td>159.0</td>
      <td>2404</td>
    </tr>
    <tr>
      <th>Australia</th>
      <td>44.0</td>
      <td>51.0</td>
      <td>20.0</td>
      <td>23.0</td>
      <td>13.0</td>
      <td>52.0</td>
      <td>35.0</td>
      <td>57.0</td>
      <td>132.0</td>
      <td>183.0</td>
      <td>157.0</td>
      <td>149.0</td>
      <td>114.0</td>
      <td>82.0</td>
      <td>1112</td>
    </tr>
  </tbody>
</table>
</div>



USA, Russia, Germany and Australia are the best countries of all time when it comes to medal tallies. What do the yearwise medal tallies look like?


```python
req_countries = ['USA', 'Russia', 'Germany', 'Australia']
row_mask_1 = medal_tally['Team'].map(lambda x: x in req_countries)

# Making a pivot table with the medal tallies
year_team_medals = pd.pivot_table(medal_tally.loc[row_mask_1, :],
                                  index = 'Year',
                                  columns = 'Team',
                                  values = 'Medal_Won',
                                  aggfunc = 'sum')

# plotting the medal tallies
year_team_medals.plot(linestyle = '-', marker = 'o', alpha = 0.9, figsize = (10,8), linewidth = 2)
xlabel('Olympic Year')
ylabel('Number of Medals')
title('Olympic Performance Comparison')
```




    Text(0.5,1,'Olympic Performance Comparison')




![png](output_59_1.png)


**Interesting Insight 1**: The blank value at 1980 for USA is not a data error! In 1980, the United States led a boycott of the Summer Olympic Games in Moscow to protest the late 1979 Soviet invasion of Afghanistan. In total, 65 nations refused to participate in the games, whereas 80 countries sent athletes to compete, India being one of those.

**Interesting Insight 2**:The missing point at 1984 for Russia is no error either! The boycott of the 1984 Summer Olympics in Los Angeles followed four years after the U.S.-led boycott of the 1980 Summer Olympics in Moscow. The boycott involved 14 Eastern Bloc countries and allies, led by the Soviet Union, which initiated the boycott on May 8, 1984.

Lets plot a breakup of medal tally by the medal type - Gold, Silver, Bronze


```python
row_mask_2 = olympics_complete_subset['Team'].map(lambda x: x in req_countries)

medal_tally_specific = pd.pivot_table(olympics_complete_subset.loc[row_mask_2,:],
                                     index = 'Team',
                                     columns = 'Medal',
                                      values = 'Medal_Won',
                                     aggfunc = 'count').drop('DNW', axis = 1)
medal_tally_specific = medal_tally_specific.loc[:, ['Gold', 'Silver', 'Bronze']]

medal_tally_specific.plot(kind = 'barh', stacked = True, figsize = (8,6))
xlabel('Number of Medals')
ylabel('Country')
```




    Text(0,0.5,'Country')




![png](output_62_1.png)


Surprisingly, countries are also in order of gold medal tallies!

### What sports are these countries best at? 
Those would be sports that they have got the most gold medals for across the years.


```python
best_team_sports = pd.pivot_table(olympics_complete_subset[row_mask_2],
                                  index = ['Team', 'Event'],
                                  columns = 'Medal',
                                  values = 'Medal_Won',
                                  aggfunc = 'sum',
                                  fill_value = 0).drop('DNW', axis = 1).sort_values(['Team', 'Gold'], 
                                                                                    ascending = [True, False]).reset_index()
best_team_sports.groupby('Team').head(5)
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
      <th>Medal</th>
      <th>Team</th>
      <th>Event</th>
      <th>Bronze</th>
      <th>Gold</th>
      <th>Silver</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Australia</td>
      <td>Hockey Women's Hockey</td>
      <td>0</td>
      <td>48</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Australia</td>
      <td>Swimming Women's 4 x 100 metres Freestyle Relay</td>
      <td>5</td>
      <td>17</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Australia</td>
      <td>Hockey Men's Hockey</td>
      <td>79</td>
      <td>16</td>
      <td>45</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Australia</td>
      <td>Swimming Women's 4 x 100 metres Medley Relay</td>
      <td>0</td>
      <td>14</td>
      <td>30</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Australia</td>
      <td>Water Polo Women's Water Polo</td>
      <td>26</td>
      <td>13</td>
      <td>0</td>
    </tr>
    <tr>
      <th>347</th>
      <td>Germany</td>
      <td>Hockey Men's Hockey</td>
      <td>32</td>
      <td>67</td>
      <td>32</td>
    </tr>
    <tr>
      <th>348</th>
      <td>Germany</td>
      <td>Rowing Men's Coxed Eights</td>
      <td>18</td>
      <td>46</td>
      <td>27</td>
    </tr>
    <tr>
      <th>349</th>
      <td>Germany</td>
      <td>Equestrianism Mixed Dressage, Team</td>
      <td>0</td>
      <td>39</td>
      <td>6</td>
    </tr>
    <tr>
      <th>350</th>
      <td>Germany</td>
      <td>Rowing Men's Quadruple Sculls</td>
      <td>8</td>
      <td>28</td>
      <td>0</td>
    </tr>
    <tr>
      <th>351</th>
      <td>Germany</td>
      <td>Rowing Women's Coxed Eights</td>
      <td>9</td>
      <td>27</td>
      <td>0</td>
    </tr>
    <tr>
      <th>699</th>
      <td>Russia</td>
      <td>Handball Men's Handball</td>
      <td>14</td>
      <td>55</td>
      <td>14</td>
    </tr>
    <tr>
      <th>700</th>
      <td>Russia</td>
      <td>Volleyball Men's Volleyball</td>
      <td>36</td>
      <td>47</td>
      <td>36</td>
    </tr>
    <tr>
      <th>701</th>
      <td>Russia</td>
      <td>Volleyball Women's Volleyball</td>
      <td>0</td>
      <td>46</td>
      <td>54</td>
    </tr>
    <tr>
      <th>702</th>
      <td>Russia</td>
      <td>Synchronized Swimming Women's Team</td>
      <td>0</td>
      <td>44</td>
      <td>0</td>
    </tr>
    <tr>
      <th>703</th>
      <td>Russia</td>
      <td>Gymnastics Women's Team All-Around</td>
      <td>6</td>
      <td>42</td>
      <td>23</td>
    </tr>
    <tr>
      <th>1053</th>
      <td>USA</td>
      <td>Basketball Men's Basketball</td>
      <td>24</td>
      <td>120</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1054</th>
      <td>USA</td>
      <td>Swimming Men's 4 x 100 metres Medley Relay</td>
      <td>0</td>
      <td>101</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1055</th>
      <td>USA</td>
      <td>Basketball Women's Basketball</td>
      <td>12</td>
      <td>95</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1056</th>
      <td>USA</td>
      <td>Swimming Men's 4 x 200 metres Freestyle Relay</td>
      <td>6</td>
      <td>73</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1057</th>
      <td>USA</td>
      <td>Swimming Women's 4 x 100 metres Medley Relay</td>
      <td>0</td>
      <td>72</td>
      <td>29</td>
    </tr>
  </tbody>
</table>
</div>



Okay, so we observe that team sports are coming at the top the most times. This would be because when a team wins gold, the records would have details of each player that won and the count would multiply times the team members.
We need to correct for this to get the actual sports that these countries are good at.

#### If a team wins more than one gold medal for an event in an edition of the olympics, then that event is a team event.


```python
identify_team_events = pd.pivot_table(olympics_complete_subset,
                                      index = ['Team', 'Year', 'Event'],
                                      columns = 'Medal',
                                      values = 'Medal_Won',
                                      aggfunc = 'sum',
                                     fill_value = 0).drop('DNW', axis = 1).reset_index()

identify_team_events = identify_team_events.loc[identify_team_events['Gold'] > 1, :]

team_sports = identify_team_events['Event'].unique()
```

So, what events are team events? The list below gives names of each event where in a single edition multiple golds were given. Going through the list, however, we found that some events have crept in which are not usually team events. Some examples include - <br>
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

    ["Canoeing Women's Kayak Doubles, 500 metres", "Rowing Men's Coxed Eights", "Baseball Men's Baseball", "Swimming Women's 4 x 100 metres Freestyle Relay", "Softball Women's Softball", "Diving Women's Synchronized Springboard", 'Tennis Mixed Doubles', "Cycling Men's Madison", "Sailing Men's Skiff", "Rowing Men's Coxed Fours", "Volleyball Men's Volleyball", "Table Tennis Men's Doubles", "Water Polo Women's Water Polo", "Badminton Men's Doubles", "Fencing Men's epee, Team", "Rowing Men's Lightweight Double Sculls", "Fencing Men's Sabre, Team", "Cycling Men's 100 kilometres Team Time Trial", "Rhythmic Gymnastics Women's Group", "Synchronized Swimming Women's Duet", "Water Polo Men's Water Polo", "Archery Women's Team", "Volleyball Women's Volleyball", "Canoeing Men's Kayak Doubles, 500 metres", "Tennis Men's Doubles", 'Sailing Mixed Skiff', "Football Women's Football", "Diving Men's Synchronized Platform", "Archery Men's Team", "Rowing Women's Coxless Pairs", "Modern Pentathlon Men's Team", "Rugby Sevens Women's Rugby Sevens", 'Sailing Mixed 5.5 metres', 'Badminton Mixed Doubles', 'Sailing Mixed Two Person Keelboat', "Diving Women's Synchronized Platform", "Badminton Women's Doubles", "Fencing Women's epee, Team", "Rowing Women's Coxless Fours", "Rowing Women's Double Sculls", "Rowing Women's Coxed Eights", "Swimming Women's 4 x 100 metres Medley Relay", "Beach Volleyball Men's Beach Volleyball", "Rowing Women's Coxed Fours", "Fencing Women's Sabre, Team", 'Sailing Mixed Two Person Heavyweight Dinghy', "Hockey Men's Hockey", 'Sailing Mixed Three Person Keelboat', 'Equestrianism Mixed Jumping, Team', "Cycling Men's Team Sprint", "Athletics Men's 4 x 400 metres Relay", "Rowing Women's Coxed Quadruple Sculls", "Rugby Sevens Men's Rugby Sevens", "Synchronized Swimming Women's Team", "Rowing Men's Double Sculls", "Diving Men's Synchronized Springboard", "Swimming Men's 4 x 200 metres Freestyle Relay", "Canoeing Men's Kayak Fours, 1,000 metres", "Rowing Men's Coxless Fours", "Rowing Men's Lightweight Coxless Fours", "Cycling Women's Team Sprint", "Athletics Men's 4 x 100 metres Relay", "Cycling Men's Team Pursuit, 4,000 metres", "Beach Volleyball Women's Beach Volleyball", "Cycling Men's Tandem Sprint, 2,000 metres", "Gymnastics Women's Team All-Around", "Sailing Women's Two Person Dinghy", "Athletics Women's 4 x 400 metres Relay", "Hockey Women's Hockey", "Sailing Women's Three Person Keelboat", "Sailing Men's Two Person Dinghy", 'Equestrianism Mixed Dressage, Team', 'Sailing Mixed Multihull', "Sailing Women's Skiff", "Handball Men's Handball", "Swimming Men's 4 x 100 metres Medley Relay", "Fencing Men's Foil, Team", "Rowing Women's Lightweight Double Sculls", "Canoeing Men's Kayak Doubles, 200 metres", "Gymnastics Men's Team All-Around", "Canoeing Men's Canadian Doubles, Slalom", "Swimming Men's 4 x 100 metres Freestyle Relay", "Canoeing Men's Canadian Doubles, 1,000 metres", "Table Tennis Men's Team", "Canoeing Men's Canadian Doubles, 500 metres", "Tennis Women's Doubles", "Rowing Men's Quadruple Sculls", 'Sailing Mixed Two Person Dinghy', "Sailing Men's Two Person Keelboat", 'Equestrianism Mixed Three-Day Event, Team', "Handball Women's Handball", "Cycling Women's Team Pursuit", "Table Tennis Women's Doubles", "Canoeing Women's Kayak Fours, 500 metres", "Fencing Women's Foil, Team", "Basketball Women's Basketball", "Rowing Men's Coxed Pairs", "Table Tennis Women's Team", "Rowing Men's Coxless Pairs", "Basketball Men's Basketball", "Rowing Women's Quadruple Sculls", "Canoeing Men's Kayak Doubles, 1,000 metres", "Football Men's Football", "Athletics Women's 4 x 100 metres Relay", "Swimming Women's 4 x 200 metres Freestyle Relay"]
    

The next thing we need to do is add a column in the dataset that correctly identifies whether the event in the given record is a team event.


```python
team_event_mask = olympics_complete_subset['Event'].map(lambda x: x in team_sports)
single_event_mask = [not i for i in team_event_mask]

medal_mask = olympics_complete_subset['Medal_Won'] == 1

olympics_complete_subset['Team_Event'] = np.where(team_event_mask & medal_mask, 1, 0)
olympics_complete_subset['Single_Event'] = np.where(single_event_mask & medal_mask, 1, 0)
olympics_complete_subset['Event_Category'] = olympics_complete_subset['Single_Event'] + olympics_complete_subset['Team_Event']

olympics_complete_subset.head()
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
      <th>index</th>
      <th>ID</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>Height</th>
      <th>Weight</th>
      <th>NOC</th>
      <th>Games</th>
      <th>Year</th>
      <th>...</th>
      <th>Event</th>
      <th>Medal</th>
      <th>Team</th>
      <th>Country Code</th>
      <th>GDP</th>
      <th>Population</th>
      <th>Medal_Won</th>
      <th>Team_Event</th>
      <th>Single_Event</th>
      <th>Event_Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1</td>
      <td>A Dijiang</td>
      <td>M</td>
      <td>24.0</td>
      <td>180.0</td>
      <td>80.0</td>
      <td>CHN</td>
      <td>1992 Summer</td>
      <td>1992</td>
      <td>...</td>
      <td>Basketball Men's Basketball</td>
      <td>DNW</td>
      <td>China</td>
      <td>CHN</td>
      <td>4.269160e+11</td>
      <td>1.164970e+09</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>2</td>
      <td>A Lamusi</td>
      <td>M</td>
      <td>23.0</td>
      <td>170.0</td>
      <td>60.0</td>
      <td>CHN</td>
      <td>2012 Summer</td>
      <td>2012</td>
      <td>...</td>
      <td>Judo Men's Extra-Lightweight</td>
      <td>DNW</td>
      <td>China</td>
      <td>CHN</td>
      <td>8.560550e+12</td>
      <td>1.350695e+09</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>31</td>
      <td>12</td>
      <td>Jyri Tapani Aalto</td>
      <td>M</td>
      <td>31.0</td>
      <td>172.0</td>
      <td>70.0</td>
      <td>FIN</td>
      <td>2000 Summer</td>
      <td>2000</td>
      <td>...</td>
      <td>Badminton Men's Singles</td>
      <td>DNW</td>
      <td>Finland</td>
      <td>FIN</td>
      <td>1.255400e+11</td>
      <td>5.176209e+06</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>32</td>
      <td>13</td>
      <td>Minna Maarit Aalto</td>
      <td>F</td>
      <td>30.0</td>
      <td>159.0</td>
      <td>55.5</td>
      <td>FIN</td>
      <td>1996 Summer</td>
      <td>1996</td>
      <td>...</td>
      <td>Sailing Women's Windsurfer</td>
      <td>DNW</td>
      <td>Finland</td>
      <td>FIN</td>
      <td>1.320990e+11</td>
      <td>5.124573e+06</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>33</td>
      <td>13</td>
      <td>Minna Maarit Aalto</td>
      <td>F</td>
      <td>34.0</td>
      <td>159.0</td>
      <td>55.5</td>
      <td>FIN</td>
      <td>2000 Summer</td>
      <td>2000</td>
      <td>...</td>
      <td>Sailing Women's Windsurfer</td>
      <td>DNW</td>
      <td>Finland</td>
      <td>FIN</td>
      <td>1.255400e+11</td>
      <td>5.176209e+06</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 23 columns</p>
</div>



Now, its time to calculate medal tally agnostic of the team size - one gold means one gold for an event. To do this we divide the number of medals by the count of winning team members. How do we get the team members? Sum of team_event column should do that for us!


```python
medal_tally_agnostic = olympics_complete_subset[row_mask_2].\
groupby(['Year', 'Team', 'Event', 'Medal'])[['Medal_Won', 'Event_Category']].\
agg('sum').reset_index()

medal_tally_agnostic['Medal_Won_Corrected'] = medal_tally_agnostic['Medal_Won']/medal_tally_agnostic['Event_Category']

medal_tally_agnostic.sort_values('Medal_Won', ascending = False).head(25)
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
      <th>Event</th>
      <th>Medal</th>
      <th>Medal_Won</th>
      <th>Event_Category</th>
      <th>Medal_Won_Corrected</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>8883</th>
      <td>2004</td>
      <td>Australia</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>24</td>
      <td>24</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>10723</th>
      <td>2008</td>
      <td>USA</td>
      <td>Baseball Men's Baseball</td>
      <td>Bronze</td>
      <td>24</td>
      <td>24</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>8601</th>
      <td>2000</td>
      <td>USA</td>
      <td>Baseball Men's Baseball</td>
      <td>Gold</td>
      <td>24</td>
      <td>24</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>7468</th>
      <td>1996</td>
      <td>USA</td>
      <td>Baseball Men's Baseball</td>
      <td>Bronze</td>
      <td>20</td>
      <td>20</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>237</th>
      <td>1964</td>
      <td>Germany</td>
      <td>Football Men's Football</td>
      <td>Bronze</td>
      <td>19</td>
      <td>19</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2028</th>
      <td>1972</td>
      <td>Russia</td>
      <td>Football Men's Football</td>
      <td>Bronze</td>
      <td>19</td>
      <td>19</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>12314</th>
      <td>2016</td>
      <td>Germany</td>
      <td>Football Women's Football</td>
      <td>Gold</td>
      <td>18</td>
      <td>18</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1807</th>
      <td>1972</td>
      <td>Germany</td>
      <td>Hockey Men's Hockey</td>
      <td>Gold</td>
      <td>18</td>
      <td>18</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>5187</th>
      <td>1988</td>
      <td>Russia</td>
      <td>Football Men's Football</td>
      <td>Gold</td>
      <td>18</td>
      <td>18</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4887</th>
      <td>1988</td>
      <td>Germany</td>
      <td>Football Men's Football</td>
      <td>Bronze</td>
      <td>18</td>
      <td>18</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1781</th>
      <td>1972</td>
      <td>Germany</td>
      <td>Football Men's Football</td>
      <td>Bronze</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>9199</th>
      <td>2004</td>
      <td>Germany</td>
      <td>Football Women's Football</td>
      <td>Bronze</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>8131</th>
      <td>2000</td>
      <td>Germany</td>
      <td>Football Women's Football</td>
      <td>Bronze</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>12313</th>
      <td>2016</td>
      <td>Germany</td>
      <td>Football Men's Football</td>
      <td>Silver</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>10020</th>
      <td>2008</td>
      <td>Australia</td>
      <td>Hockey Men's Hockey</td>
      <td>Bronze</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>10265</th>
      <td>2008</td>
      <td>Germany</td>
      <td>Hockey Men's Hockey</td>
      <td>Gold</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>12333</th>
      <td>2016</td>
      <td>Germany</td>
      <td>Hockey Women's Hockey</td>
      <td>Bronze</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>10794</th>
      <td>2008</td>
      <td>USA</td>
      <td>Football Women's Football</td>
      <td>Gold</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>3763</th>
      <td>1980</td>
      <td>Russia</td>
      <td>Football Men's Football</td>
      <td>Bronze</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>9740</th>
      <td>2004</td>
      <td>USA</td>
      <td>Football Women's Football</td>
      <td>Gold</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2656</th>
      <td>1976</td>
      <td>Germany</td>
      <td>Football Men's Football</td>
      <td>Gold</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>11834</th>
      <td>2012</td>
      <td>USA</td>
      <td>Football Women's Football</td>
      <td>Gold</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>3493</th>
      <td>1980</td>
      <td>Germany</td>
      <td>Football Men's Football</td>
      <td>Silver</td>
      <td>17</td>
      <td>17</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>5930</th>
      <td>1992</td>
      <td>Germany</td>
      <td>Hockey Women's Hockey</td>
      <td>Silver</td>
      <td>16</td>
      <td>16</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>5929</th>
      <td>1992</td>
      <td>Germany</td>
      <td>Hockey Men's Hockey</td>
      <td>Gold</td>
      <td>16</td>
      <td>16</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>



To get the sports, teams are best at, we now aggregate the medal_tally_agnostic dataframe as we did earlier.


```python
best_team_sports = pd.pivot_table(medal_tally_agnostic,
                                  index = ['Team', 'Event'],
                                  columns = 'Medal',
                                  values = 'Medal_Won_Corrected',
                                  aggfunc = 'sum',
                                  fill_value = 0).sort_values(['Team', 'Gold'], ascending = [True, False]).reset_index()

best_team_sports.groupby('Team').head(5)
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
      <th>Medal</th>
      <th>Team</th>
      <th>Event</th>
      <th>Bronze</th>
      <th>DNW</th>
      <th>Gold</th>
      <th>Silver</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Australia</td>
      <td>Swimming Men's 1,500 metres Freestyle</td>
      <td>4</td>
      <td>0</td>
      <td>5</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Australia</td>
      <td>Swimming Men's 400 metres Freestyle</td>
      <td>3</td>
      <td>0</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Australia</td>
      <td>Equestrianism Mixed Three-Day Event, Team</td>
      <td>3</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Australia</td>
      <td>Hockey Women's Hockey</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Australia</td>
      <td>Sailing Men's Two Person Dinghy</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>347</th>
      <td>Germany</td>
      <td>Equestrianism Mixed Dressage, Team</td>
      <td>0</td>
      <td>0</td>
      <td>11</td>
      <td>2</td>
    </tr>
    <tr>
      <th>348</th>
      <td>Germany</td>
      <td>Canoeing Women's Kayak Doubles, 500 metres</td>
      <td>2</td>
      <td>0</td>
      <td>7</td>
      <td>4</td>
    </tr>
    <tr>
      <th>349</th>
      <td>Germany</td>
      <td>Rowing Men's Quadruple Sculls</td>
      <td>2</td>
      <td>0</td>
      <td>7</td>
      <td>0</td>
    </tr>
    <tr>
      <th>350</th>
      <td>Germany</td>
      <td>Rowing Women's Quadruple Sculls</td>
      <td>1</td>
      <td>0</td>
      <td>6</td>
      <td>1</td>
    </tr>
    <tr>
      <th>351</th>
      <td>Germany</td>
      <td>Athletics Men's Discus Throw</td>
      <td>2</td>
      <td>0</td>
      <td>5</td>
      <td>4</td>
    </tr>
    <tr>
      <th>699</th>
      <td>Russia</td>
      <td>Wrestling Men's Heavyweight, Freestyle</td>
      <td>0</td>
      <td>0</td>
      <td>9</td>
      <td>1</td>
    </tr>
    <tr>
      <th>700</th>
      <td>Russia</td>
      <td>Fencing Men's Sabre, Team</td>
      <td>1</td>
      <td>0</td>
      <td>7</td>
      <td>2</td>
    </tr>
    <tr>
      <th>701</th>
      <td>Russia</td>
      <td>Gymnastics Women's Team All-Around</td>
      <td>1</td>
      <td>0</td>
      <td>7</td>
      <td>4</td>
    </tr>
    <tr>
      <th>702</th>
      <td>Russia</td>
      <td>Rhythmic Gymnastics Women's Individual</td>
      <td>3</td>
      <td>0</td>
      <td>7</td>
      <td>4</td>
    </tr>
    <tr>
      <th>703</th>
      <td>Russia</td>
      <td>Wrestling Men's Light-Heavyweight, Freestyle</td>
      <td>2</td>
      <td>0</td>
      <td>7</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1053</th>
      <td>USA</td>
      <td>Swimming Men's 4 x 100 metres Medley Relay</td>
      <td>0</td>
      <td>0</td>
      <td>13</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1054</th>
      <td>USA</td>
      <td>Swimming Men's 4 x 200 metres Freestyle Relay</td>
      <td>1</td>
      <td>0</td>
      <td>11</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1055</th>
      <td>USA</td>
      <td>Athletics Men's 4 x 400 metres Relay</td>
      <td>0</td>
      <td>0</td>
      <td>10</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1056</th>
      <td>USA</td>
      <td>Athletics Men's 400 metres</td>
      <td>7</td>
      <td>0</td>
      <td>10</td>
      <td>8</td>
    </tr>
    <tr>
      <th>1057</th>
      <td>USA</td>
      <td>Basketball Men's Basketball</td>
      <td>2</td>
      <td>0</td>
      <td>10</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



Much Better! This now presents a better picture of the sports competencies of these nations! What we have done is essentially consider one win in a team event in one edition mean one medal. So what the above table shows us is how many times does a country win repeatedly at an event! Note that Basketball was in the top for USA even before and it is still at number 5!

### Next we look at the size of the olympic contingent that these countries send to the Olympics!


```python
# Get year wise team wise athletes.
year_team_athelete = olympics_complete_subset.loc[row_mask_2, ['Year','Team', 'Name']].drop_duplicates()

# sum these up to get total contingent size.
contingent_size = pd.pivot_table(year_team_athelete,
                                 index = 'Year',
                                 columns = 'Team',
                                 values = 'Name',
                                 aggfunc = 'count')

fig, (ax1, ax2) = subplots(nrows = 1,
                          ncols = 2,
                          sharex = True,
                          figsize = (18,7))

contingent_size.plot(ax = ax1, linestyle = '-', marker = 'o', linewidth = 2)
ax1.plot(1972, contingent_size.loc[1972, 'Germany'], marker = '^', color = 'orange', ms = 14)
ax1.plot(1980, contingent_size.loc[1980, 'Russia'], marker = '^', color = 'green', ms = 14)
ax1.plot(1984, contingent_size.loc[1984, 'USA'], marker = '^', color = 'red', ms = 14)
ax1.plot(2000, contingent_size.loc[2000, 'Australia'], marker = '^', color = 'blue', ms = 14)
ax1.set_xlabel('Olympic Year')
ax1.set_ylabel('Number of Athletes')
ax1.set_title('Contingent Size Per Year')

year_team_medals.plot(ax = ax2, linestyle = '-', marker = 'o', linewidth = 2)
ax2.plot(1972, year_team_medals.loc[1972, 'Germany'], marker = '^', color = 'orange', ms = 14)
ax2.plot(1980, year_team_medals.loc[1980, 'Russia'], marker = '^', color = 'green', ms = 14)
ax2.plot(1984, year_team_medals.loc[1984, 'USA'], marker = '^', color = 'red', ms = 14)
ax2.plot(2000, year_team_medals.loc[2000, 'Australia'], marker = '^', color = 'blue', ms = 14)
ax2.set_xlabel('Olympic Year')
ax2.set_ylabel('Number of Medals')
ax2.set_title('Olympic Medal Tally')
```




    Text(0.5,1,'Olympic Medal Tally')




![png](output_78_1.png)


It is interesting to see that for each of these countries, a point of peak in the contingent size translates directly to a peak in the medal tally! These have been marked as large triangles on the plots

### Are fit athletes the reason for these countries medal tally?


```python

```




    721.0



### Is home advantage a thing?
Do countries win more when they are playing at home?

### Are there some commonalities between the countries?
Are there common things they are all good at?


```python
team_commonalities = best_team_sports.merge(olympics_complete_subset.loc[:,['Sport', 'Event']].drop_duplicates(),
                                           left_on = 'Event',
                                           right_on = 'Event')

team_commonalities = team_commonalities.sort_values(['Team', 'Gold'], ascending = [True, False])
team_commonalities = team_commonalities.groupby('Team').head(10).reset_index()

pd.pivot_table(team_commonalities,
              index = 'Sport',
              columns = 'Team',
              values = 'Event',
              aggfunc = 'count',
              fill_value = 0,
              margins = True).sort_values('All', ascending = False)[1:]
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
      <th>Team</th>
      <th>Australia</th>
      <th>Germany</th>
      <th>Russia</th>
      <th>USA</th>
      <th>All</th>
    </tr>
    <tr>
      <th>Sport</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Swimming</th>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>5</td>
      <td>10</td>
    </tr>
    <tr>
      <th>Athletics</th>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>4</td>
      <td>8</td>
    </tr>
    <tr>
      <th>Canoeing</th>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
    </tr>
    <tr>
      <th>Wrestling</th>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>4</td>
    </tr>
    <tr>
      <th>Rowing</th>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>Cycling</th>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>Equestrianism</th>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>Basketball</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Fencing</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Gymnastics</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Hockey</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Rhythmic Gymnastics</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Sailing</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Weightlifting</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



#### Interesting Insights 3:
1. One thing all nations have in their top 10 sports is athletics!
2. Australia and US have a penchant for swimming events!
3. Germany is solid in rowing and canoeing!
4. Russia loves to wrestle!


```python
identify_team_events[identify_team_events['Event'] == "Baseball Men's Baseball"]
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
      <th>Medal</th>
      <th>Team</th>
      <th>Year</th>
      <th>Event</th>
      <th>Bronze</th>
      <th>Gold</th>
      <th>Silver</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>16465</th>
      <td>Cuba</td>
      <td>1992</td>
      <td>Baseball Men's Baseball</td>
      <td>0</td>
      <td>20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16559</th>
      <td>Cuba</td>
      <td>1996</td>
      <td>Baseball Men's Baseball</td>
      <td>0</td>
      <td>20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16777</th>
      <td>Cuba</td>
      <td>2004</td>
      <td>Baseball Men's Baseball</td>
      <td>0</td>
      <td>24</td>
      <td>0</td>
    </tr>
    <tr>
      <th>60016</th>
      <td>South Korea</td>
      <td>2008</td>
      <td>Baseball Men's Baseball</td>
      <td>0</td>
      <td>24</td>
      <td>0</td>
    </tr>
    <tr>
      <th>71539</th>
      <td>USA</td>
      <td>2000</td>
      <td>Baseball Men's Baseball</td>
      <td>0</td>
      <td>24</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# To check data by filtering rows!

event_mask = olympics_complete_subset['Event'] == "Baseball Men's Baseball"
team_mask = olympics_complete_subset['Team'] == "Australia"
year_mask = olympics_complete_subset['Year'] == 2004
medal_mask = olympics_complete_subset['Medal_Won'] == 1

olympics_complete_subset.loc[event_mask & year_mask & medal_mask & team_mask, :]
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
      <th>index</th>
      <th>ID</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>Height</th>
      <th>Weight</th>
      <th>NOC</th>
      <th>Games</th>
      <th>Year</th>
      <th>...</th>
      <th>Event</th>
      <th>Medal</th>
      <th>Team</th>
      <th>Country Code</th>
      <th>GDP</th>
      <th>Population</th>
      <th>Medal_Won</th>
      <th>Team_Event</th>
      <th>Single_Event</th>
      <th>Event_Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4628</th>
      <td>6780</td>
      <td>3803</td>
      <td>Craig Anthony Anderson</td>
      <td>M</td>
      <td>23.0</td>
      <td>187.0</td>
      <td>86.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17333</th>
      <td>29085</td>
      <td>15041</td>
      <td>Thomas Robert "Tom" Brice</td>
      <td>M</td>
      <td>22.0</td>
      <td>195.0</td>
      <td>98.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>19294</th>
      <td>32495</td>
      <td>16704</td>
      <td>Adrian Mark Burnside</td>
      <td>M</td>
      <td>27.0</td>
      <td>193.0</td>
      <td>100.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>41115</th>
      <td>69486</td>
      <td>35410</td>
      <td>Gavin Fingleson</td>
      <td>M</td>
      <td>28.0</td>
      <td>177.0</td>
      <td>87.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>48431</th>
      <td>81976</td>
      <td>41627</td>
      <td>Paul Gonzalez</td>
      <td>M</td>
      <td>35.0</td>
      <td>186.0</td>
      <td>86.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>69806</th>
      <td>119317</td>
      <td>60409</td>
      <td>Nicholas Andrew "Nick" Kimpton</td>
      <td>M</td>
      <td>20.0</td>
      <td>185.0</td>
      <td>85.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>69922</th>
      <td>119544</td>
      <td>60516</td>
      <td>Brendan Robert Kingman</td>
      <td>M</td>
      <td>31.0</td>
      <td>186.0</td>
      <td>112.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>79822</th>
      <td>137706</td>
      <td>69216</td>
      <td>Craig Edward Lewis</td>
      <td>M</td>
      <td>27.0</td>
      <td>195.0</td>
      <td>100.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>81948</th>
      <td>141177</td>
      <td>70852</td>
      <td>Graeme John Lloyd</td>
      <td>M</td>
      <td>37.0</td>
      <td>200.0</td>
      <td>108.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>100747</th>
      <td>171902</td>
      <td>86379</td>
      <td>David Wayne "Dave" Nilsson</td>
      <td>M</td>
      <td>34.0</td>
      <td>193.0</td>
      <td>110.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>102615</th>
      <td>175243</td>
      <td>88067</td>
      <td>Trent Carl Wayne Oeltjen</td>
      <td>M</td>
      <td>21.0</td>
      <td>188.0</td>
      <td>88.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>104976</th>
      <td>179289</td>
      <td>90086</td>
      <td>Wayne Daniel Ough</td>
      <td>M</td>
      <td>25.0</td>
      <td>190.0</td>
      <td>95.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>105197</th>
      <td>179603</td>
      <td>90257</td>
      <td>Chris Andrew Oxspring</td>
      <td>M</td>
      <td>27.0</td>
      <td>184.0</td>
      <td>82.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>119486</th>
      <td>203968</td>
      <td>102405</td>
      <td>Brett Nicholas Roneberg</td>
      <td>M</td>
      <td>25.0</td>
      <td>185.0</td>
      <td>93.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>120107</th>
      <td>205214</td>
      <td>103042</td>
      <td>Ryan Benjamin Rowland Smith</td>
      <td>M</td>
      <td>21.0</td>
      <td>190.0</td>
      <td>108.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>133639</th>
      <td>229009</td>
      <td>114982</td>
      <td>John M. Stephens</td>
      <td>M</td>
      <td>24.0</td>
      <td>182.0</td>
      <td>84.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>134000</th>
      <td>229671</td>
      <td>115321</td>
      <td>Phillip Matthew "Phil" Stockman</td>
      <td>M</td>
      <td>24.0</td>
      <td>200.0</td>
      <td>100.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>137383</th>
      <td>235742</td>
      <td>118198</td>
      <td>Brett Paul Tamburrino</td>
      <td>M</td>
      <td>22.0</td>
      <td>178.0</td>
      <td>88.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>139433</th>
      <td>239293</td>
      <td>119948</td>
      <td>Richard Graeme Thompson</td>
      <td>M</td>
      <td>20.0</td>
      <td>184.0</td>
      <td>80.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>144094</th>
      <td>246950</td>
      <td>123643</td>
      <td>Andrew Utting</td>
      <td>M</td>
      <td>26.0</td>
      <td>187.0</td>
      <td>98.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>144728</th>
      <td>248111</td>
      <td>124217</td>
      <td>Rodney Van Buizen</td>
      <td>M</td>
      <td>23.0</td>
      <td>180.0</td>
      <td>92.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>151254</th>
      <td>260420</td>
      <td>130321</td>
      <td>Ben Wigmore</td>
      <td>M</td>
      <td>22.0</td>
      <td>184.0</td>
      <td>85.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>151566</th>
      <td>260980</td>
      <td>130599</td>
      <td>Glenn David Williams</td>
      <td>M</td>
      <td>27.0</td>
      <td>188.0</td>
      <td>90.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>151580</th>
      <td>260997</td>
      <td>130610</td>
      <td>Jeffrey Francis "Jeff" Williams</td>
      <td>M</td>
      <td>32.0</td>
      <td>183.0</td>
      <td>84.0</td>
      <td>AUS</td>
      <td>2004 Summer</td>
      <td>2004</td>
      <td>...</td>
      <td>Baseball Men's Baseball</td>
      <td>Silver</td>
      <td>Australia</td>
      <td>AUS</td>
      <td>6.133300e+11</td>
      <td>20127400.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>24 rows × 23 columns</p>
</div>


