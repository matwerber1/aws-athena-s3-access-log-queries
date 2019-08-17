# aws-athena-s3-access-log-queries

A collection of SQL queries that may be used with Amazon S3 server access logs. 

## View summary of all S3 API calls in last 10 days

The values of the **operation** column correspond with the API operations listed at:
https://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketOps.html

```sql
SELECT 
  bucket
  ,operation
  ,key
  -- ,requesturi_key                  -- uncomment to see additional query params 
  ,httpstatus
  ,errorcode
  -- ,requester                       -- uncomment to see who made the API call
  ,count(*) AS count
FROM   
  mybucket_logs
WHERE     
  parse_datetime(requestdatetime,'dd/MMM/yyyy:HH:mm:ss Z')
    >= (CURRENT_DATE - interval '10' day)
GROUP BY
  bucket
  ,operation
  ,key
  -- ,requesturi_key                   -- uncomment to see additional query params
  ,httpstatus
  ,errorcode
  -- ,requester                        -- uncomment to see who made the API call
ORDER BY  
  key
```

## View calls to s3::ListBucket in the last 10 days

Note that an **operation** of a `REST.GET.BUCKET` corresponds to the s3:ListBucket API as documented in:
https://docs.aws.amazon.com/AmazonS3/latest/API/v2-RESTBucketGET.html

```sql
SELECT 
  bucket
  ,operation
  key
  -- ,requesturi_key                  -- uncomment to see additional query params 
  ,httpstatus
  ,errorcode
  -- ,requester                       -- uncomment to see who made the API call
  ,count(*) AS count
FROM   
  mybucket_logs
WHERE     
  operation = 'REST.GET.BUCKET'
GROUP BY
  bucket
  ,operation 
  ,key
  -- ,requesturi_key                   -- uncomment to see additional query params
  ,httpstatus
  ,errorcode
  -- ,requester                        -- uncomment to see who made the API call
ORDER BY  
  key
```
