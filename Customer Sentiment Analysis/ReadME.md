## Cord Cutting
![Cord Cutting](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/Customer_Sentiment_Analysis/CordCutting.jpg)

### The Task
Use reddit data to figure out the tendencies of cord-cutters: how they compare streaming services, what their concerns are with cable tv, their favorite shows and complaints with their streaming services.

### The data
Using the Reddit API and [pushshift.io](https://pushshift.io/) (Big Query), we extracted user comments from the [cordcutter subreddit](https://www.reddit.com/r/cordcutters/) for the last 4 months. We had a total of about 30,000 comments to analyse.

### Concepts used
* Lift analysis and MDS plots
* Finding Relevant entities (Cable companies, streaming services) using the Spacy package.
* Topic modelling using Latent Dirichlet Allocation
* Extracting bigrams using POS tagging, Sentiment Mining and Semantic Orientation

### Results
* LDA shows important topics that people discuss are sports packages, net neutrality, the traditional cable experience, and user experience.
* Niche products – FuboTV – are associated more with product related topics as opposed to general interest topics.
* POS Bigram analysis reveals interesting crowd attitudes towards streaming services, specific shows and actors. For instance, 'Adam Sandler' and 'Big Bang Theory' are associated with negative sentiments while 'Stranger Things' are associated with positive sentiments.

### Repository Structure
'Cord_Cutting.ipynb' has the code with the final analysis. <br>
['Cord_Cutting.pdf'](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Customer%20Sentiment%20Analysis/Cord_Cutting.pdf) has our final presentation.
