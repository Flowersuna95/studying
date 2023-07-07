# GA4 & Bigquery

First, the streaming export of GA4 data allows near real-time data to be processed in the intraday table of your BigQuery dataset. 
By default, GA4 exports in daily batches, meaning you won’t be able to access today’s data until it’s exported sometime in the following day.

Second, the default table expiration of 60 days means that you will never be able to query past the last two months’ of data. 
For some, this is enough, but for many it might be prudent to change this expiration. 
The trick is that you need to change the default expiration as soon as possible. 
Any tables created before the default expiration is updated will keep their original expiration.

You can fix this by manually updating every single existing table to the new expiration, but this can be extremely tedious if you have hundreds of tables to update.

So remember to check and modify (if necessary) the default table expiration from the dataset settings as well as for any table that has already been created in the dataset.


### Key Takeaway #1
The BigQuery Sandbox lets you test the service at no cost. 
You might still want to upgrade soon, as there are certain benefits that are only available once you upgrade away from the sandbox.

### Key Takeaway #2
To link a Google Analytics 4 property with Google BigQuery, you need a Google Cloud Project for the data export to work. 
Make sure you select the data storage region carefully when establishing the link.

### Key Takeaway #3
Additional configurations you might want to consider include enabling the streaming data export for almost real-time data in the intraday table, and modifying the default table expiration to prevent the data from being deleted from BigQuery in exactly 60 days from ingestion.
