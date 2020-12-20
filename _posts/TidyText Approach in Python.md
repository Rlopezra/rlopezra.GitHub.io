During my tenure at Google Wear OS, one of my main sources of data that I had to analyze was open ended responses from surveys. The responses were usually submitted by users through the Happiness Tracking Survey (HaTS), a bug report, or a survey for a feature in development. Some of the most common questions users were asked are:
- What do you like the **MOST** about your device?
- What do you like the **LEAST** about your device?
- Why did you **NOT** use the feature?
- What would (did) you expect to happen when you used the feature?
- Do you have anything else to say about the feature?

One of the most popular methods to analyze open-ended data is to categorize the responses with a code that encapsulate it. When categorizing open-ended responses, UX Researchers usually had a pre-determined list of codes or would add codes to a list as they categorize responses. Taking this approach to analyze open-ended responses is a simple and effective method when dealing with small sample sizes (less than 50 responses) and few open-ended questions.

However, categorizing open-ended responses is not a feasible method when dealing with a large data set or when time is of the essence. This is where R’s TidyText package comes in. The TidyText package and its companion book, [Text Mining with R](https://www.tidytextmining.com/index.html), makes analyzing open ended responses easy with its tokenization function that represents the data in a tabular format to easily summarize and visualize the tokens. The TidyText package also has a built in TF-IDF function that calculates a token’s TD-IDF based on a categorical column(s). Furthermore, the accompanying book is a great resource for any Data Scientists/Data Analyst/Research Analyst that has to analyze text-based data and report out to stakeholders. For example, Figure 4.1 in the book illustrates the most important bigrams in each Jane Austen book using TD-IDF. It is a simple graph to understand but conveys a lot of meaningful information. 

![Image](https://www.tidytextmining.com/04-word-combinations_files/figure-html/bigramtfidf-1.png)


Prior to adapting these methods, Wear OS analyzed large data set of open-ended responses by taking a random sample of responses and manually coding them. The methods taught by the book allowed me to analyze entire datasets to find out what where users liked and disliked about their Wear OS devices. I replicated many of the graphs in Text Mining with R using Wear OS' data and they were received with great fanfare.

So when I tried to apply these methods in Python, I was dishearten to find that the language had no equivalent package to calculate a token’s TF-IDF. So, I created one. 

Full disclosure, I used this post as a [reference](https://sigdelta.com/blog/text-analysis-in-pandas/). 


### Calculating TF-IDF the Tidy Way

For this post, I'll be using [Kaggle's 515k Hotel Reviews Data in Europe](https://www.kaggle.com/jiashenliu/515k-hotel-reviews-data-in-europe). I've already tokenized the Positive Reviews and removed all stopwords. 


```python
from ast import literal_eval
import pandas as pd
import numpy as np
import seaborn as sns
%matplotlib inline

df = pd.read_csv("C:/Users/rlope/Documents/Data for Projects/Hotel/clean hotel reviews.csv", converters={'pos_tokens': literal_eval})

df
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
      <th>Hotel_Address</th>
      <th>Review_Date</th>
      <th>Average_Score</th>
      <th>Hotel_Name</th>
      <th>Reviewer_Nationality</th>
      <th>Totl_Num_Rev</th>
      <th>Positive_Review</th>
      <th>Rev_Tot_Pos_WC</th>
      <th>Revi_Score</th>
      <th>Tags</th>
      <th>pos_tokens</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>s Gravesandestraat 55 Oost 1092 AA Amsterdam ...</td>
      <td>8/3/2017</td>
      <td>7.7</td>
      <td>Hotel Arena</td>
      <td>Russia</td>
      <td>1403</td>
      <td>Only the park outside of the hotel was beauti...</td>
      <td>11</td>
      <td>2.9</td>
      <td>[' Leisure trip ', ' Couple ', ' Duplex Double...</td>
      <td>[park, outside, hotel, beautiful]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>s Gravesandestraat 55 Oost 1092 AA Amsterdam ...</td>
      <td>8/3/2017</td>
      <td>7.7</td>
      <td>Hotel Arena</td>
      <td>Ireland</td>
      <td>1403</td>
      <td>No real complaints the hotel was great great ...</td>
      <td>105</td>
      <td>7.5</td>
      <td>[' Leisure trip ', ' Couple ', ' Duplex Double...</td>
      <td>[real, complaint, hotel, great, great, locatio...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>s Gravesandestraat 55 Oost 1092 AA Amsterdam ...</td>
      <td>7/31/2017</td>
      <td>7.7</td>
      <td>Hotel Arena</td>
      <td>Australia</td>
      <td>1403</td>
      <td>Location was good and staff were ok It is cut...</td>
      <td>21</td>
      <td>7.1</td>
      <td>[' Leisure trip ', ' Family with young childre...</td>
      <td>[location, good, staff, ok, cute, hotel, break...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>s Gravesandestraat 55 Oost 1092 AA Amsterdam ...</td>
      <td>7/31/2017</td>
      <td>7.7</td>
      <td>Hotel Arena</td>
      <td>United Kingdom</td>
      <td>1403</td>
      <td>Great location in nice surroundings the bar a...</td>
      <td>26</td>
      <td>3.8</td>
      <td>[' Leisure trip ', ' Solo traveler ', ' Duplex...</td>
      <td>[great, location, nice, surroundings, bar, res...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>s Gravesandestraat 55 Oost 1092 AA Amsterdam ...</td>
      <td>7/24/2017</td>
      <td>7.7</td>
      <td>Hotel Arena</td>
      <td>New Zealand</td>
      <td>1403</td>
      <td>Amazing location and building Romantic setting</td>
      <td>8</td>
      <td>6.7</td>
      <td>[' Leisure trip ', ' Couple ', ' Suite ', ' St...</td>
      <td>[amazing, location, building, romantic, setting]</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>479787</th>
      <td>Wurzbachgasse 21 15 Rudolfsheim F nfhaus 1150 ...</td>
      <td>9/6/2015</td>
      <td>8.1</td>
      <td>Atlantis Hotel Vienna</td>
      <td>Kuwait</td>
      <td>2823</td>
      <td>helpful staff allowed me to check in early as...</td>
      <td>18</td>
      <td>10.0</td>
      <td>[' Leisure trip ', ' Family with young childre...</td>
      <td>[helpful, staff, allowed, check, early, arrive...</td>
    </tr>
    <tr>
      <th>479788</th>
      <td>Wurzbachgasse 21 15 Rudolfsheim F nfhaus 1150 ...</td>
      <td>8/30/2015</td>
      <td>8.1</td>
      <td>Atlantis Hotel Vienna</td>
      <td>Kuwait</td>
      <td>2823</td>
      <td>location</td>
      <td>2</td>
      <td>7.0</td>
      <td>[' Leisure trip ', ' Family with older childre...</td>
      <td>[location]</td>
    </tr>
    <tr>
      <th>479789</th>
      <td>Wurzbachgasse 21 15 Rudolfsheim F nfhaus 1150 ...</td>
      <td>8/22/2015</td>
      <td>8.1</td>
      <td>Atlantis Hotel Vienna</td>
      <td>Estonia</td>
      <td>2823</td>
      <td>Breakfast was ok and we got earlier check in</td>
      <td>11</td>
      <td>5.8</td>
      <td>[' Leisure trip ', ' Family with young childre...</td>
      <td>[breakfast, ok, got, earlier, check]</td>
    </tr>
    <tr>
      <th>479790</th>
      <td>Wurzbachgasse 21 15 Rudolfsheim F nfhaus 1150 ...</td>
      <td>8/17/2015</td>
      <td>8.1</td>
      <td>Atlantis Hotel Vienna</td>
      <td>Mexico</td>
      <td>2823</td>
      <td>The rooms are enormous and really comfortable...</td>
      <td>25</td>
      <td>8.8</td>
      <td>[' Leisure trip ', ' Group ', ' Standard Tripl...</td>
      <td>[room, enormous, really, comfortable, believe,...</td>
    </tr>
    <tr>
      <th>479791</th>
      <td>Wurzbachgasse 21 15 Rudolfsheim F nfhaus 1150 ...</td>
      <td>8/9/2015</td>
      <td>8.1</td>
      <td>Atlantis Hotel Vienna</td>
      <td>Hungary</td>
      <td>2823</td>
      <td>staff was very kind</td>
      <td>6</td>
      <td>8.3</td>
      <td>[' Leisure trip ', ' Family with young childre...</td>
      <td>[staff, kind]</td>
    </tr>
  </tbody>
</table>
<p>479792 rows × 11 columns</p>
</div>



After loading the dataset, I'll create a copy of the dataframe and explode the tokens column so that each token has its own record. 


```python
df_copy = df.copy()
df_explode = df_copy.explode('pos_tokens')
df_explode
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
      <th>Hotel_Address</th>
      <th>Review_Date</th>
      <th>Average_Score</th>
      <th>Hotel_Name</th>
      <th>Reviewer_Nationality</th>
      <th>Totl_Num_Rev</th>
      <th>Positive_Review</th>
      <th>Rev_Tot_Pos_WC</th>
      <th>Revi_Score</th>
      <th>Tags</th>
      <th>pos_tokens</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>s Gravesandestraat 55 Oost 1092 AA Amsterdam ...</td>
      <td>8/3/2017</td>
      <td>7.7</td>
      <td>Hotel Arena</td>
      <td>Russia</td>
      <td>1403</td>
      <td>Only the park outside of the hotel was beauti...</td>
      <td>11</td>
      <td>2.9</td>
      <td>[' Leisure trip ', ' Couple ', ' Duplex Double...</td>
      <td>park</td>
    </tr>
    <tr>
      <th>0</th>
      <td>s Gravesandestraat 55 Oost 1092 AA Amsterdam ...</td>
      <td>8/3/2017</td>
      <td>7.7</td>
      <td>Hotel Arena</td>
      <td>Russia</td>
      <td>1403</td>
      <td>Only the park outside of the hotel was beauti...</td>
      <td>11</td>
      <td>2.9</td>
      <td>[' Leisure trip ', ' Couple ', ' Duplex Double...</td>
      <td>outside</td>
    </tr>
    <tr>
      <th>0</th>
      <td>s Gravesandestraat 55 Oost 1092 AA Amsterdam ...</td>
      <td>8/3/2017</td>
      <td>7.7</td>
      <td>Hotel Arena</td>
      <td>Russia</td>
      <td>1403</td>
      <td>Only the park outside of the hotel was beauti...</td>
      <td>11</td>
      <td>2.9</td>
      <td>[' Leisure trip ', ' Couple ', ' Duplex Double...</td>
      <td>hotel</td>
    </tr>
    <tr>
      <th>0</th>
      <td>s Gravesandestraat 55 Oost 1092 AA Amsterdam ...</td>
      <td>8/3/2017</td>
      <td>7.7</td>
      <td>Hotel Arena</td>
      <td>Russia</td>
      <td>1403</td>
      <td>Only the park outside of the hotel was beauti...</td>
      <td>11</td>
      <td>2.9</td>
      <td>[' Leisure trip ', ' Couple ', ' Duplex Double...</td>
      <td>beautiful</td>
    </tr>
    <tr>
      <th>1</th>
      <td>s Gravesandestraat 55 Oost 1092 AA Amsterdam ...</td>
      <td>8/3/2017</td>
      <td>7.7</td>
      <td>Hotel Arena</td>
      <td>Ireland</td>
      <td>1403</td>
      <td>No real complaints the hotel was great great ...</td>
      <td>105</td>
      <td>7.5</td>
      <td>[' Leisure trip ', ' Couple ', ' Duplex Double...</td>
      <td>real</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>479790</th>
      <td>Wurzbachgasse 21 15 Rudolfsheim F nfhaus 1150 ...</td>
      <td>8/17/2015</td>
      <td>8.1</td>
      <td>Atlantis Hotel Vienna</td>
      <td>Mexico</td>
      <td>2823</td>
      <td>The rooms are enormous and really comfortable...</td>
      <td>25</td>
      <td>8.8</td>
      <td>[' Leisure trip ', ' Group ', ' Standard Tripl...</td>
      <td>member</td>
    </tr>
    <tr>
      <th>479790</th>
      <td>Wurzbachgasse 21 15 Rudolfsheim F nfhaus 1150 ...</td>
      <td>8/17/2015</td>
      <td>8.1</td>
      <td>Atlantis Hotel Vienna</td>
      <td>Mexico</td>
      <td>2823</td>
      <td>The rooms are enormous and really comfortable...</td>
      <td>25</td>
      <td>8.8</td>
      <td>[' Leisure trip ', ' Group ', ' Standard Tripl...</td>
      <td>comfy</td>
    </tr>
    <tr>
      <th>479790</th>
      <td>Wurzbachgasse 21 15 Rudolfsheim F nfhaus 1150 ...</td>
      <td>8/17/2015</td>
      <td>8.1</td>
      <td>Atlantis Hotel Vienna</td>
      <td>Mexico</td>
      <td>2823</td>
      <td>The rooms are enormous and really comfortable...</td>
      <td>25</td>
      <td>8.8</td>
      <td>[' Leisure trip ', ' Group ', ' Standard Tripl...</td>
      <td>space</td>
    </tr>
    <tr>
      <th>479791</th>
      <td>Wurzbachgasse 21 15 Rudolfsheim F nfhaus 1150 ...</td>
      <td>8/9/2015</td>
      <td>8.1</td>
      <td>Atlantis Hotel Vienna</td>
      <td>Hungary</td>
      <td>2823</td>
      <td>staff was very kind</td>
      <td>6</td>
      <td>8.3</td>
      <td>[' Leisure trip ', ' Family with young childre...</td>
      <td>staff</td>
    </tr>
    <tr>
      <th>479791</th>
      <td>Wurzbachgasse 21 15 Rudolfsheim F nfhaus 1150 ...</td>
      <td>8/9/2015</td>
      <td>8.1</td>
      <td>Atlantis Hotel Vienna</td>
      <td>Hungary</td>
      <td>2823</td>
      <td>staff was very kind</td>
      <td>6</td>
      <td>8.3</td>
      <td>[' Leisure trip ', ' Family with young childre...</td>
      <td>kind</td>
    </tr>
  </tbody>
</table>
<p>4690239 rows × 11 columns</p>
</div>



To calculate a token's TF-IDF, we need two pieces of information:

- Term Frequency (TF) :How token's frequency within each document
- Inverse Document Frequency (IDF): The inverse of how many documents the token is in

For our example, we will be calculating token's TD-IDF based on 'Reviewer Nationality.'

The first step is calculate tokens' term frequency for each nationality and the total wordcount for each nationality


```python
#getting word count in each group
counts = df_explode.groupby('Reviewer_Nationality')['pos_tokens'].value_counts().to_frame().rename(columns={'pos_tokens':'n_w'})

#each group's total word count
word_sum = counts.groupby(level=0).sum().rename(columns={'n_w': 'n_d'})
```

The next step is to join the two pieces of information and calcuate 'TF'


```python
#adding each group's total word count to word frequency
tf = counts.join(word_sum)
    
tf['tf'] = tf.n_w/tf.n_d

tf
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
      <th></th>
      <th>n_w</th>
      <th>n_d</th>
      <th>tf</th>
    </tr>
    <tr>
      <th>Reviewer_Nationality</th>
      <th>pos_tokens</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">Abkhazia Georgia</th>
      <th>room</th>
      <td>63</td>
      <td>1533</td>
      <td>0.041096</td>
    </tr>
    <tr>
      <th>hotel</th>
      <td>50</td>
      <td>1533</td>
      <td>0.032616</td>
    </tr>
    <tr>
      <th>staff</th>
      <td>48</td>
      <td>1533</td>
      <td>0.031311</td>
    </tr>
    <tr>
      <th>location</th>
      <td>42</td>
      <td>1533</td>
      <td>0.027397</td>
    </tr>
    <tr>
      <th>good</th>
      <td>31</td>
      <td>1533</td>
      <td>0.020222</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">Zimbabwe</th>
      <th>westminister</th>
      <td>1</td>
      <td>398</td>
      <td>0.002513</td>
    </tr>
    <tr>
      <th>wharf</th>
      <td>1</td>
      <td>398</td>
      <td>0.002513</td>
    </tr>
    <tr>
      <th>wi</th>
      <td>1</td>
      <td>398</td>
      <td>0.002513</td>
    </tr>
    <tr>
      <th>window</th>
      <td>1</td>
      <td>398</td>
      <td>0.002513</td>
    </tr>
    <tr>
      <th>worked</th>
      <td>1</td>
      <td>398</td>
      <td>0.002513</td>
    </tr>
  </tbody>
</table>
<p>262172 rows × 3 columns</p>
</div>



And now we have the 'TD' of 'TD-IDF'

For 'IDF', we first have to figure out the number of nationalities that used each token. 


```python
#total number of nationalities
c_d = counts.index.get_level_values(0).nunique()

#calculating the inverse document frequency aka the number of nationalities that used that token
idf = counts.reset_index().groupby('pos_tokens')['Reviewer_Nationality'].nunique().to_frame().rename(columns={'Reviewer_Nationality':'i_d'}).sort_values('i_d')
```

The next step is to diveide c_d by the inverse document frequency and taking the log.


```python
idf['idf'] = np.log(c_d/idf.i_d.values)

idf
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
      <th>i_d</th>
      <th>idf</th>
    </tr>
    <tr>
      <th>pos_tokens</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>zzzzzxx</th>
      <td>1</td>
      <td>5.407172</td>
    </tr>
    <tr>
      <th>turbo</th>
      <td>1</td>
      <td>5.407172</td>
    </tr>
    <tr>
      <th>lacaster</th>
      <td>1</td>
      <td>5.407172</td>
    </tr>
    <tr>
      <th>lac</th>
      <td>1</td>
      <td>5.407172</td>
    </tr>
    <tr>
      <th>labyrinthine</th>
      <td>1</td>
      <td>5.407172</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>hotel</th>
      <td>187</td>
      <td>0.176063</td>
    </tr>
    <tr>
      <th>good</th>
      <td>195</td>
      <td>0.134172</td>
    </tr>
    <tr>
      <th>location</th>
      <td>205</td>
      <td>0.084162</td>
    </tr>
    <tr>
      <th>room</th>
      <td>205</td>
      <td>0.084162</td>
    </tr>
    <tr>
      <th>staff</th>
      <td>212</td>
      <td>0.050585</td>
    </tr>
  </tbody>
</table>
<p>47445 rows × 2 columns</p>
</div>



Now we have the 'IDF'. 

The last step is to join the 'TF' and 'IDF' dataframes and multiply 'TF' and 'IDF' to calculate 'TF-IDF'


```python
#Joining idf dataframe to tf dataframe
tf_idf = tf.join(idf)

#Calculating TF-IDF by multiplying TF and IDF
tf_idf['tf_idf'] = tf_idf.tf * tf_idf.idf
    
tf_idf = tf_idf.reset_index()

tf_idf
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
      <th>Reviewer_Nationality</th>
      <th>pos_tokens</th>
      <th>n_w</th>
      <th>n_d</th>
      <th>tf</th>
      <th>i_d</th>
      <th>idf</th>
      <th>tf_idf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abkhazia Georgia</td>
      <td>room</td>
      <td>63</td>
      <td>1533</td>
      <td>0.041096</td>
      <td>205</td>
      <td>0.084162</td>
      <td>0.003459</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Abkhazia Georgia</td>
      <td>hotel</td>
      <td>50</td>
      <td>1533</td>
      <td>0.032616</td>
      <td>187</td>
      <td>0.176063</td>
      <td>0.005742</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Abkhazia Georgia</td>
      <td>staff</td>
      <td>48</td>
      <td>1533</td>
      <td>0.031311</td>
      <td>212</td>
      <td>0.050585</td>
      <td>0.001584</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Abkhazia Georgia</td>
      <td>location</td>
      <td>42</td>
      <td>1533</td>
      <td>0.027397</td>
      <td>205</td>
      <td>0.084162</td>
      <td>0.002306</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Abkhazia Georgia</td>
      <td>good</td>
      <td>31</td>
      <td>1533</td>
      <td>0.020222</td>
      <td>195</td>
      <td>0.134172</td>
      <td>0.002713</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>262167</th>
      <td>Zimbabwe</td>
      <td>westminister</td>
      <td>1</td>
      <td>398</td>
      <td>0.002513</td>
      <td>7</td>
      <td>3.461262</td>
      <td>0.008697</td>
    </tr>
    <tr>
      <th>262168</th>
      <td>Zimbabwe</td>
      <td>wharf</td>
      <td>1</td>
      <td>398</td>
      <td>0.002513</td>
      <td>51</td>
      <td>1.475346</td>
      <td>0.003707</td>
    </tr>
    <tr>
      <th>262169</th>
      <td>Zimbabwe</td>
      <td>wi</td>
      <td>1</td>
      <td>398</td>
      <td>0.002513</td>
      <td>89</td>
      <td>0.918535</td>
      <td>0.002308</td>
    </tr>
    <tr>
      <th>262170</th>
      <td>Zimbabwe</td>
      <td>window</td>
      <td>1</td>
      <td>398</td>
      <td>0.002513</td>
      <td>90</td>
      <td>0.907362</td>
      <td>0.002280</td>
    </tr>
    <tr>
      <th>262171</th>
      <td>Zimbabwe</td>
      <td>worked</td>
      <td>1</td>
      <td>398</td>
      <td>0.002513</td>
      <td>77</td>
      <td>1.063366</td>
      <td>0.002672</td>
    </tr>
  </tbody>
</table>
<p>262172 rows × 8 columns</p>
</div>



Awesome! Now we have a dataframe that resembles the TidyText approach. 

For simplicity purposes, lets take a look at United States and Australias top 10 most frequently and most important used words.

Lets create a barplot that shows the most frequently used words for both nationalities.


```python
countries = ['United States of America', 'Australia']

comparison = tf_idf[tf_idf['Reviewer_Nationality'].isin(countries)]

top_tf = comparison.sort_values(['Reviewer_Nationality','tf'],ascending=False).groupby('Reviewer_Nationality').head(10)

sns.catplot(x="tf", y="pos_tokens", col="Reviewer_Nationality",data=top_tf, kind="bar", orient="h").set_xticklabels(rotation=45).set_axis_labels("Most Frequently Used","Word")
```




    <seaborn.axisgrid.FacetGrid at 0x21c95431b00>




![png](output_16_1.png)


Now lets plot 'TD-IDF' to show each nationalities most important words. 


```python
top_tfidf = comparison.sort_values(['Reviewer_Nationality','tf_idf'],ascending=False).groupby('Reviewer_Nationality').head(10)
sns.catplot(x="tf_idf", y="pos_tokens", col="Reviewer_Nationality",data=top_tfidf, kind="bar", orient="h").set_xticklabels(rotation=45).set_axis_labels("Most Important","Word")
```




    <seaborn.axisgrid.FacetGrid at 0x21c18f93748>




![png](output_18_1.png)


A cool graph that we can create is a scatterplot that compares how frequently a token is used by each nationality. The way to interpret this graph is words that are along the diagonal line are equally used by each nationality. The further the token deviates from the diagonal line, the more it used by a nationality. In our example, words that are above the diagonal line are more frequently used by Americans.

**Footnote:** I usually plot this graph using Plotly or visulation tool like Tableau, PowerBI or Google Data Studio to allow stakeholders to explore the dataset. This graph is not very informative when all of the points overlap and you can't discern them.


```python
sub_set_ct = comparison.pivot_table(index='pos_tokens',columns='Reviewer_Nationality', values='tf').reset_index()

sns.regplot(x='Australia', y='United States of America', data=sub_set_ct, ci=None)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x21c9617b780>




![png](output_20_1.png)


### Reproducibility

Calculating 'TD-IDF' isn't difficulty, but it can be tedious if you have to repeat the process several times. So lets create a function to simplify the process.


```python
def td_idf(data, group, token):
    
    #getting word count in each group
    counts = data.groupby(group)[token].value_counts().to_frame().rename(columns={token:'n_w'})
    
    #each group's total word count
    word_sum = counts.groupby(level=0).sum().rename(columns={'n_w': 'n_d'})
    
    #adding each group's total word count to word frequency
    tf = counts.join(word_sum)
    
    tf['tf'] = tf.n_w/tf.n_d
    
    c_d = counts.index.get_level_values(0).nunique()
    
    idf = counts.reset_index().groupby(token)[group].nunique().to_frame().rename(columns={group:'i_d'}).sort_values('i_d')
    
    idf['idf'] = np.log(c_d/idf.i_d.values)
    #idf = idf.reset_index()
    
    tf_idf = tf.join(idf)
    #tf_idf = tf.merge(idf, on=token, how='left')

    tf_idf['tf_idf'] = tf_idf.tf * tf_idf.idf
    
    tf_idf = tf_idf.reset_index()
    
    return tf_idf
```


```python
hotel_comp = td_idf(df_explode,'Hotel_Name', 'pos_tokens' )
hotel_comp
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
      <th>Hotel_Name</th>
      <th>pos_tokens</th>
      <th>n_w</th>
      <th>n_d</th>
      <th>tf</th>
      <th>i_d</th>
      <th>idf</th>
      <th>tf_idf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11 Cadogan Gardens</td>
      <td>staff</td>
      <td>88</td>
      <td>1587</td>
      <td>0.055451</td>
      <td>1492</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11 Cadogan Gardens</td>
      <td>location</td>
      <td>60</td>
      <td>1587</td>
      <td>0.037807</td>
      <td>1490</td>
      <td>0.001341</td>
      <td>0.000051</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11 Cadogan Gardens</td>
      <td>room</td>
      <td>45</td>
      <td>1587</td>
      <td>0.028355</td>
      <td>1490</td>
      <td>0.001341</td>
      <td>0.000038</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11 Cadogan Gardens</td>
      <td>hotel</td>
      <td>40</td>
      <td>1587</td>
      <td>0.025205</td>
      <td>1490</td>
      <td>0.001341</td>
      <td>0.000034</td>
    </tr>
    <tr>
      <th>4</th>
      <td>11 Cadogan Gardens</td>
      <td>friendly</td>
      <td>30</td>
      <td>1587</td>
      <td>0.018904</td>
      <td>1491</td>
      <td>0.000670</td>
      <td>0.000013</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1035432</th>
      <td>pentahotel Vienna</td>
      <td>wonderland</td>
      <td>1</td>
      <td>1098</td>
      <td>0.000911</td>
      <td>52</td>
      <td>3.356629</td>
      <td>0.003057</td>
    </tr>
    <tr>
      <th>1035433</th>
      <td>pentahotel Vienna</td>
      <td>work</td>
      <td>1</td>
      <td>1098</td>
      <td>0.000911</td>
      <td>911</td>
      <td>0.493330</td>
      <td>0.000449</td>
    </tr>
    <tr>
      <th>1035434</th>
      <td>pentahotel Vienna</td>
      <td>working</td>
      <td>1</td>
      <td>1098</td>
      <td>0.000911</td>
      <td>687</td>
      <td>0.775538</td>
      <td>0.000706</td>
    </tr>
    <tr>
      <th>1035435</th>
      <td>pentahotel Vienna</td>
      <td>worth</td>
      <td>1</td>
      <td>1098</td>
      <td>0.000911</td>
      <td>886</td>
      <td>0.521156</td>
      <td>0.000475</td>
    </tr>
    <tr>
      <th>1035436</th>
      <td>pentahotel Vienna</td>
      <td>wos</td>
      <td>1</td>
      <td>1098</td>
      <td>0.000911</td>
      <td>3</td>
      <td>6.209260</td>
      <td>0.005655</td>
    </tr>
  </tbody>
</table>
<p>1035437 rows × 8 columns</p>
</div>



Awesome! Now we have a function that takes in the dataset, the column we are comparing, and the tokens to calculate the 'TF-IDF.'


```python

```
