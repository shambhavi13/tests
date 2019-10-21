---
Intial Preprocessing
---
The dataset provided is massive in size (11GB) with around 21M rows and 41 columns. Preprocessing steps is very important to remove unused columns. Furthermore, it is a good approach to initially work with a chunk of the dataset before using the dataset in entirety. Following are the steps undertaken to pre-process the dataset:

1. Use Dask: Its a open source library and is very flexible for parallel computing
2. Using pandas chunk size it basically loads the file in smaller chunks.

After initial review of the data, following pre-processing steps were executed:
1. Curtail the length of zipcode attribute to length of 5 characters (Truncate everything to length 5)
2. Set zipcodes `00000` to `NaN`s
3. Retain the digits and remove alphabets/words form the ‘incident zip’ column
4. Clean ‘complain type’ column and remove all unwanted characters including blank spaces
5. Change the data type of the ‘incident zip’ to string

---
Question 1: What are the number of complaints per department for each zip code?
---
The approach is to use dataframe groupby function to calculate the total countof complaint type per department and incident zip.

---
Question 2: Anomalous complaints for department for each zip code?
---
The approach I employed was to set a numeric threshold to identify anomalous complaints. Since complaint type is categorical, it can be standard to characterize by percentage.To solve this calculated the total number of complaints and total count of complaints per department for each department. then we calculate percentage threshold can be 25th and 75th percentile
```coffeescript
Q1 = df.Percentage.quantile(0.25)
Q3 = df.Percentage.quantile(0.75)
IQR = Q3 - Q1
```
After we got the IQR score, we can find the anomalous complaints

```coffeescript
print(df['complaint_type'] < (Q1 - 1.5 * IQR)) |(df['complaint_type'] > (Q3 + 1.5 * IQR))
```
----
Question 3: For each zipcode find which other zipcode they are most and least similar
to. Does this change over time? Is so, how
----
To find the similarities between zipcodes, firstly, the top 10 complaint types were identified. Among these, "Noise" is the top complaint. Next, the few top ‘incident zips’ were identified which have the most complaints of this kind. 
```coffeescript
	incident_zip	count
0	11226           63189
1	10031	        60765
2	10467	        54756
```
In the above results snapshot we can say that incident zip 11226 is most similar to 10031 because the complaint count is closer.

Moving forward, two analysis have been conducted:

a). **_Based on geographic location (Cluster data)_**:
For a given complaint type, a cluster graph has been plotted to demonstrate the concentration of complaints with respect to the latitude and longitude of the incident. It can be seen from the data that the incidents seem to be clustering heavily in two factions of geographic location and provide a trend for the zip codes. It may be that, in this case, the noise complaints come largely for some defined neighborhoods due to their own social/economic reasons.

b). **_Based on incident date_**:
This intends to observe that for a given complaint type and zip code, has the number of incidents drastically changed after 4 years (in this case 2011 and 2015). The following table shows the trend of how the incident count has changed for any zip code between 2011 and 2015.

```coffeescript
    incident_zip	2011_count	2015_count
1	11226	        2313	    6307
2	10032	        2090	    5301
```
From the above bar plot the number of noise complaints have drastically increased over year( 2011 to 2015) but that similarity between zip codes hasn't changed. The zipcode 11226 and 10032 still closer in terms of noise complaint counts

---
Question 4: What factors affect the time to close a ticket?
---
For this question we need to identify relevant features by exploring the dataset 
and extracting the most interesting properties that reveal somehow clear correlations
1. Extract relevant complaint type from the main dataset
2. Time series analysis: we can try to focus on <br>
    -- period of the year (frequency of complaints over the year) <br>
    -- time of day <br>
    -- resolution time clear comparison over the years (this is a key point of this analysis
       to provide a measure of how much has the 311 efficiency improved/got worse over the years)

for predicting factors to close the ticket: Is a supervised learning classification problem
1. Predictor variable : Month, Incident zip
2. Target variable: resolution time in terms of number of days calculated 
by difference between closed and created date and then divide this into number of
buckets 
```coffeescript
Bucket 1: Less than two days
Bucket 2: 2-6 days
```
Hence our target variable will have 2 classes. Then we can use RandomForestClassifier

---
Question 5: Come up with three interesting questions you would ask of the data.  For each
question, briefly discuss how you would answer the question and what other
datasets would be necessary. Pick one and experiment with it!
---

One of the intriguing aspects could be to understand how any social or economic factors 
influence the nature of incidents. For example, do gas leaks happen more often due to old houses an 
low income neighbourhood (economic issue), or if crime is concentrated in particular zip codes more 
than other because of the gentry of the neighbourhood (social aspect). 
To study this, a set of chief complain types should be identified and then date be further investigated.

Another could time of year and its correlation with the type of complaint. 
For example, snow related complaints should generally be registered in Dec-March time frame



