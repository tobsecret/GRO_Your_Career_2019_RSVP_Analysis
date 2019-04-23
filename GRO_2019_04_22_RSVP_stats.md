

```python
import altair as alt
import pandas as pd
%load_ext watermark
%watermark -iv -v -d
```

    pandas 0.24.2
    altair 2.4.1
    2019-04-22 
    
    CPython 3.7.3
    IPython 7.4.0


## Aims
The aim of this notebook is to analyze our RSVP data and see which schools I have to encourage to send more e-mails to their listservs. This RSVP data was collected until the night before the conference which is on the 23rd of April, 2019.
We are using pandas for data wrangling and altair for plotting.


```python
df = pd.read_csv('rsvp_data.csv')
df['time'] = pd.to_datetime(df['time'])
df.head()
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
      <th>rsvp_no</th>
      <th>time</th>
      <th>institution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2019-03-07 16:58:00</td>
      <td>Non-academic Institution</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2019-03-14 19:11:00</td>
      <td>Other</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2019-03-18 14:28:00</td>
      <td>Non-academic Institution</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2019-03-19 16:33:00</td>
      <td>CUNY</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>2019-03-19 16:45:00</td>
      <td>CUNY</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['counts'] = df.groupby('institution').cumcount()+1
```


```python
alt.Chart(df).properties(width=600).mark_line().encode(
    alt.Color('institution',
        scale=alt.Scale(scheme='category20')),
    x='time',
    y='counts',
    tooltip='institution'
).interactive()
```




![png](GRO_2019_04_22_RSVP_stats_files/GRO_2019_04_22_RSVP_stats_4_0.png)



We used this information around April 11th to motivate additional advertising efforts by Columbia and other schools. 
What we had noticed was that after weeks of advertising, one clickbaity e-mail at NYU still brought in a huge upwards spike in RSVPs (around 10th of April), meaning that the common fear of over-exposing a target audience to ads would diminish their willingness to RSVP for the conference. 

We can tell that `NYU` was bringing the most RSVPs on April 11th, but we the colleagues at `Columbia` were able to close the gap. One also has to consider that `CUNY` was split up into different labels late in the process. If one were to add the CUNY RSVPs up towards the end, they would fall between Weill Cornell and Einstein. Additionally we see quite a few RSVPs for `Non-academic Institution`, emphasizing that at our conference you don't just have the panelists and speakers to network with, but also your fellow attendees from industry.

### When were the biggest spikes in signups/ e-mail blasts

The aim here is to have some fun with the numbers to estimate when each university received e-mail blasts, by looking at the differential in signups over time. 

We will make use of the following solution, which attempts to compute the local gradient from incoherently sampled data:

https://stackoverflow.com/questions/41780489/python-pandas-how-to-calculate-derivative-gradient


```python
dummy = pd.concat((df['rsvp_no'].diff()/ df['time'].diff().dt.total_seconds(), df['time']), axis=1).rename(columns={0:'differential'})
dummy.head()
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
      <th>differential</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>2019-03-07 16:58:00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.000002</td>
      <td>2019-03-14 19:11:00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.000003</td>
      <td>2019-03-18 14:28:00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.000011</td>
      <td>2019-03-19 16:33:00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.001389</td>
      <td>2019-03-19 16:45:00</td>
    </tr>
  </tbody>
</table>
</div>



Let's look at all RSVPs together:


```python
alt.Chart(dummy).properties(width=600).mark_line().encode(
    x='time',
    y='differential'
)
```




![png](GRO_2019_04_22_RSVP_stats_files/GRO_2019_04_22_RSVP_stats_9_0.png)



Now let's look at how this same pictures looks when we calculate these data by institution:


```python
differentials =  df.groupby('institution').apply(
    lambda group: pd.concat([group['time'], group['counts'].diff()/ df['time'].diff().dt.total_seconds()], axis=1)
).reset_index().dropna().rename(columns={0:'differential'})\
.loc[:,['institution', 'time', 'differential']]
differentials.head()
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
      <th>institution</th>
      <th>time</th>
      <th>differential</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>CUNY</td>
      <td>2019-03-19 16:45:00</td>
      <td>0.001389</td>
    </tr>
    <tr>
      <th>5</th>
      <td>CUNY</td>
      <td>2019-03-20 12:58:00</td>
      <td>0.000014</td>
    </tr>
    <tr>
      <th>6</th>
      <td>CUNY</td>
      <td>2019-03-24 23:46:00</td>
      <td>0.000003</td>
    </tr>
    <tr>
      <th>7</th>
      <td>CUNY</td>
      <td>2019-03-24 23:48:00</td>
      <td>0.008333</td>
    </tr>
    <tr>
      <th>8</th>
      <td>CUNY</td>
      <td>2019-03-25 07:25:00</td>
      <td>0.000036</td>
    </tr>
  </tbody>
</table>
</div>




```python
alt.Chart(differentials).properties(width=800).mark_line().encode(
    alt.Color('institution',
        scale=alt.Scale(scheme='category20')),
    x='time',
    y='differential',
    tooltip='institution'

)
```




![png](GRO_2019_04_22_RSVP_stats_files/GRO_2019_04_22_RSVP_stats_12_0.png)



This is not terribly helpful looking, the only thing we can tell from this is that there are not necessarily many more bursts of NYU signups. 

It would probably be a lot easier to read if we used a windowed count of RSVPs. So let's try counting the number of RSVPs in a 2h window: 


```python
rsvp_2h_avg_institution = df.groupby('institution').apply(             # we group by institution
    lambda frame: frame.set_index('time').counts.rolling('2H').count() # for every institution, we want to do a rolling average of unique RSVPs with a 2h time window
).reset_index(                                                         # resetting the index, so it's easier to consume for altair
)
rsvp_2h_avg_institution.groupby('institution').head().head(10)         # showing that the counts make sense - if time points are 2h apart, then they are counted together
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
      <th>institution</th>
      <th>time</th>
      <th>counts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CUNY</td>
      <td>2019-03-19 16:33:00</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CUNY</td>
      <td>2019-03-19 16:45:00</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CUNY</td>
      <td>2019-03-20 12:58:00</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CUNY</td>
      <td>2019-03-24 23:46:00</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CUNY</td>
      <td>2019-03-24 23:48:00</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>36</th>
      <td>CUNY (Baruch)</td>
      <td>2019-04-22 10:59:00</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>37</th>
      <td>CUNY (CCNY)</td>
      <td>2019-04-16 19:02:00</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>38</th>
      <td>CUNY (CCNY)</td>
      <td>2019-04-17 18:19:00</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>39</th>
      <td>CUNY (CCNY)</td>
      <td>2019-04-18 06:35:00</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>40</th>
      <td>CUNY (CCNY)</td>
      <td>2019-04-18 07:14:00</td>
      <td>2.0</td>
    </tr>
  </tbody>
</table>
</div>



So let's plot these windowed counts:


```python
alt.Chart(rsvp_2h_avg_institution).properties(width=600).mark_line().encode(
    x='time',
    y='counts',
    color = alt.Color('institution',
        scale=alt.Scale(scheme='category20')), 
    tooltip='institution'

).interactive()
```




![png](GRO_2019_04_22_RSVP_stats_files/GRO_2019_04_22_RSVP_stats_16_0.png)



We can pretty clearly see that there is a burst of peaks from Columbia in the last two weeks and if you look clearly, there was another burst right around March 25th, which is when we all started advertising.
NYU, only has a couple of peaks but major activity at each of those peaks. This could be tied to a larger audience (the PhD program at NYU is one of the larger ones in the country), stronger receptiveness for career development opportunities, or more effective marketing.

You may also ask why we have RSVPs from before then and that's because some of our members already RSVPed for the conference when the website was not public yet.

Let's look at these counts institution by institution:


```python
alt.Chart(rsvp_2h_avg_institution).mark_line().encode(
    x='time',
    y='counts',
    row='institution'
)

```




![png](GRO_2019_04_22_RSVP_stats_files/GRO_2019_04_22_RSVP_stats_18_0.png)



We can pretty clearly tell from this that Mount Sinai and Einstein had similar peak patterns to NYU, as did Columbia. However Columbia seems to have also additionally ramped up advertising efforts in the last two weeks before the conference. TRI-I (Weill Cornell, Rockefeller and Memorial Sloan Kettering) show reduced number of RSVP peaks, which is mostly because of multiple other career events that happened in the marketing period and would have clashed with advertising for the local events, plus the organizers of those events were of course busy with organizing their own events. 

### What time of the day do people sign up?

Conventional wisdom has it that people mostly respond to e-mails in the morning but what time of day do people sign up the most?

We do have to take into account here that our results are going to be biased around when we sent e-mails but it might still be interesting to look at.


```python
hourly_rsvps_institution = df.groupby(['institution', df.time.dt.hour]).size().reset_index().rename(columns={0:'counts'})
hourly_rsvps_institution.head()
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
      <th>institution</th>
      <th>time</th>
      <th>counts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CUNY</td>
      <td>7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CUNY</td>
      <td>9</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CUNY</td>
      <td>10</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CUNY</td>
      <td>11</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CUNY</td>
      <td>12</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




```python
alt.Chart(hourly_rsvps_institution).properties(width=600)\
.mark_line()\
.encode(
    x='time',
    y='counts',
    color=alt.Color('institution',
        scale=alt.Scale(scheme='category20')),
    tooltip='institution'
).interactive()
```




![png](GRO_2019_04_22_RSVP_stats_files/GRO_2019_04_22_RSVP_stats_22_0.png)




```python
alt.Chart(hourly_rsvps_institution).properties(width=600)\
.mark_area()\
.encode(
    x='time',
    y=alt.Y('counts', stack='zero'),
    color=alt.Color('institution',
        scale=alt.Scale(scheme='category20')),
    tooltip='institution'
).interactive()
```




![png](GRO_2019_04_22_RSVP_stats_files/GRO_2019_04_22_RSVP_stats_23_0.png)



This shows that we got most of our RSVPs around 10 am. It is however important to keep in mind that:

a) RSVPs are usually do not necessarily have to be following an e-mail instantly, i.e. getting most RSVPs at 10 am means the best time to e-mail could be before then.

b) We tried to e-mail around 9 am, so our data collection is fundamentally biased. 

There is a second hump in the afternoon, because we would often send out an e-mail after lunch if we did not get to it before lunch. Some of the RSVPs during the nightly hours are likely by folks on conferences abroad or otherwise. What is somewhat surprising is the steady amouind of RSVPs from 8 pm to midnight - many folks checking their e-mails late at night.

Any of these hypothesis would need to be tested with varying e-mail timing randomly across a cohort. 

Even then it would be difficult to assess because if we randomly subselect for example NYU students, some of the recipients of the earlier e-mails could talk to their colleagues who receive later e-mails, enhancing the effectiveness of later e-mails.

## Summary

We have learned
* we can use RSVP data to analyse the response of our different communities
* repeated e-mails do not substantially lower the effectiveness of advertising (we don't generally e-mail more than once per day)
* repeated e-mail blasts over short periods of time may be highly effective (Columbia, 2 weeks before conference)
* peak response is at 10 am but likely due to when we usually send our e-mails
