# Creating-a-Streaming-Data-Pipeline-for-a-Real-Time-Dashboard-with-Cloud-Dataflow
Create a BigQuery dataset

Option 1: Command Line

Open Cloud Shell and run the below command to create the taxirides dataset

bq mk taxirides
Run this command to create the taxirides.realtime table (empty schema we will stream into later)
bq mk \
--time_partitioning_field timestamp \
--schema ride_id:string,point_idx:integer,latitude:float,longitude:float,\
timestamp:timestamp,meter_reading:float,meter_increment:float,ride_status:string,\
passenger_count:integer -t taxirides.realtime
Option 2: BigQuery Console UI
Skip these steps if you created the tables using the command line

In the GCP Console, go to Navigation menu> BigQuery.

Once there, click on your GCP Project ID from the left-hand menu

Now on the right-hand side of the console underneath the query editor click CREATE DATASET.

Give the new dataset the name taxirides, leave all the other fields the way they are, and click Create dataset.

If you look at the left-hand resources menu, you should see your newly created dataset

Click on the taxirides dataset

Click create table

Name the table realtime

For schema, click edit as text and paste in the below

ride_id:string,
point_idx:integer,
latitude:float,
longitude:float,
timestamp:timestamp,
meter_reading:float,
meter_increment:float,
ride_status:string,
passenger_count:integer
Under Partition and cluster settings, select the timestamp option for the Partitioning field.

Confirm against the below screenshot:

bq-taxi-table.png

Click the Create Table button.
Create a Cloud Storage Bucket
Skip this step if you already have a bucket created

Cloud Storage allows world-wide storage and retrieval of any amount of data at any time. You can use Cloud Storage for a range of scenarios including serving website content, storing data for archival and disaster recovery, or distributing large data objects to users via direct download. In this lab we will use Cloud Storage to provide working space for our Cloud Dataflow pipeline.

In the GCP Console, go to Navigation menu > Storage.
Click CREATE BUCKET.
For Name, paste in your GCP project ID.
For Default storage class, click Multi-regional if it is not already selected.
For Location, choose the selection closest to you.
Click Create.
Set up a Cloud Dataflow Pipeline
Cloud Dataflow is a serverless way to carry out data analysis. In this lab, you will set up a streaming data pipeline to read sensor data from Pub/Sub, compute the maximum temperature within a time window, and write this out to BigQuery.

In the GCP Console, go to Navigation menu > Dataflow.

In the top menu bar, click CREATE JOB FROM TEMPLATE.

Enter streaming-taxi-pipeline as the Job name for your Cloud Dataflow job.

Under Dataflow template, select the Cloud Pub/Sub Topic to BigQuery template.

Under Cloud Pub/Sub input topic, enter projects/pubsub-public-data/topics/taxirides-realtime

Under BigQuery output table, enter <myprojectid>:taxirides.realtime

Note: there is a colon : between the project and dataset name and a dot . between the dataset and table name. Also, make sure to replace <myprojectid> with your assigned project Id.

Under Temporary Location, enter gs://<mybucket>/tmp/ and replace <mybucket> with your bucket name.

Click the Run job button.

A new streaming job has started! You can now see a visual representation of the data pipeline.

Job.png

Analyze the Taxi Data Using BigQuery
To analyze the data as it is streaming:

In the GCP Console, open the Navigation menu and select BigQuery.

Enter the following query in the Query editor and click RUN:

SELECT * FROM taxirides.realtime LIMIT 10
If no records are returned, wait another minute and re-run the above query (Dataflow takes 3-5 minutes to setup the stream). You will receive a similar output:
output.png

Perform aggregations on the stream for reporting
Copy and paste the below query and run


WITH streaming_data AS (

SELECT
  timestamp,
  TIMESTAMP_TRUNC(timestamp, HOUR, 'UTC') AS hour,
  TIMESTAMP_TRUNC(timestamp, MINUTE, 'UTC') AS minute,
  TIMESTAMP_TRUNC(timestamp, SECOND, 'UTC') AS second,
  ride_id,
  latitude,
  longitude,
  meter_reading,
  ride_status,
  passenger_count
FROM
  taxirides.realtime
WHERE ride_status = 'dropoff'
ORDER BY timestamp DESC
LIMIT 100000

)

# calculate aggregations on stream for reporting:
SELECT
 ROW_NUMBER() OVER() AS dashboard_sort,
 minute,
 COUNT(DISTINCT ride_id) AS total_rides,
 SUM(meter_reading) AS total_revenue,
 SUM(passenger_count) AS total_passengers
FROM streaming_data
GROUP BY minute, timestamp

The result shows key metrics by the minute for every taxi drop-off

Create a Real-Time Dashboard
Click Explore Data and then, select Explore with Data Studio.
On the Welcome page, click on GET STARTED.
On the Next page, click Authorize.
Note: If you are getting the prompt Oops… Not able to connect to your data then click Back. Click Save in save data studio explorer.
Click on GET STARTED and acknowledge the Terms of Service. Click Accept.

Select No, thanks for all in preferences and click Done.

Refresh the tab to load the data.

Specify the below settings:

Chart type: column chart
Date range dimension: dashboard_sort
Drill down: dashboard_sort (Make sure that Drill down option is turned ON.)
Dimension: dashboard_sort, minute
Metric: SUM() total_rides, SUM() total_passengers, SUM() total_revenue (If Record Count is present then, mouse over Record Count and click the (x) to remove it.)
Sort: dashboard_sort Ascending (latest rides first)
chart.png

Note: Visualizing data at a minute-level granularity is currently not supported in Data Studio as a timestamp. This is why we created our own dashboard_sort dimension.

When you're happy with your dashboard, click Save to save this data source
If prompted then select following,
On the Welcome page, click on GET STARTED.

On the Terms page, click on the checkbox to acknowledge the terms and click ACCEPT.

On the Preferences page, select No, thanks for each option to receive email notifications, and click DONE.

Whenever anyone visits your dashboard, it will be up-to-date with the latest transactions. You can try it yourself by clicking on the Refresh button near the Save button

Stop the Cloud Dataflow job
Navigate back to Cloud Dataflow

Click the streaming-taxi-pipeline

Click Stop Job and Cancel pipeline

This will free up resources for your project

Congratulations!
In this lab you Pub/Sub to collect streaming data messages from Taxis and feed it through your Dataflow pipeline into BigQuery.

