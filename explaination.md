---
Intial Preprocessing
---
The dataset provided is massive. Preprocessing steps is very important to reduce 
dimension or remove unused columns. Below are the few steps I tried

1. Use Dask: Its a open source libaray and very flexible for parallel computing
2. Using pandas chunksize it basically loads the file in smaller chuSnks.

After initial look at the data
1. Stick with zipcode length to 5 (Truncate everything to length 5)
2. Set zipcodes `00000` to `nans`
3. Keep only digits and remove letters and words form incident zip column
4. Clean complain type column remove all unwanted characters
5. change incident zip dtype to str

---
Question 1: What are the number of complaints per department for each zip code?
---
The approach is to use dataframe groupby function to calculate the total count
of complaint type per department and incident zip.

---
Question 2: Anolmous complaints for department for each zip code?
---
The approach I took was to set threshold to identify analmous complaints. Since 
complaint type is categorical it can be standard to characterize by percentage.
To solve this calculated the total number of complaints and total count of complaints
per department for each department. then we calculate percentage threshold can be 
25th and 75th percentile
```coffeescript
Q1 = df.Percentage.quantile(0.25)
Q3 = df.Percentage.quantile(0.75)
IQR = Q3 - Q1
```
After we got the IQR score, we can find the anolmous complaints

```coffeescript
print(df['complaint_type'] < (Q1 - 1.5 * IQR)) |(df['complaint_type'] > (Q3 + 1.5 * IQR))
```
------
Question 3: For each zipcode find which other zipcode they are most and least similar
to. Does this change over time? Is so, how
-------
To find the similarities between zipcode we identified top 10 complaints. In this
dataset "Noise" is top complaints after that we can calculate which incident zips
have most noise complaints. 
```coffeescript
	incident_zip	count
0	11226           63189
1	10031	        60765
2	10467	        54756
```
Looking at the above results we can say that incident zip 11226 is most similar to 
10031 because the complaint count is closer.

Other analysis would be to see if there are some zips where some specific complaints 
tend to happen more than what's typical.

For analysing over time extracted the year from `created_date` and `closed_date` 
to measure the similarity over year and if that change with time.

```coffeescript
	incident_zip	count	created_year
	11226.0	        36045	2011
	11226.0	        36045	2015

	10031.0	        33002	2011
	10031.0	        33002	2015
```
From above results we can see that complaint count doesn't change over year that much

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


