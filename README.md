# aws-athena-s3-access-log-queries

If you've enabled [Amazon S3 server access logging](https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerLogs.html) on a given bucket, S3 will output access logs on that bucket to another bucket of your choice. From there, you can easily query these logs using Amazon Athena, Amazon Redshift Spectrum, Amazon EMR, and other tools. 

This repo contains example queries for Amazon Athena. 

## Prerequisites

Enable server access logging on one or more of your Amazon S3 buckets and create a corresponding table table in your Athena / Glue data catalog by following the guide below: 
https://aws.amazon.com/premiumsupport/knowledge-center/analyze-logs-athena/

# Example SQL

## List different operation types in S3 access logs

The S3 access logs have an **operation** field that corresponds to the REST APIs listed at: 
https://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketOps.html

Based on my anecdotal testing, the values of **operation** are in the format of `REST.XXX.YYYY`, where `XXX` is the HTTP method and `YYYY` is the S3 resource / configuration being acted upon. 

You can run this statement to see the operations that have been performed on your bucket: 
```sql
SELECT DISTINCT
  operation
FROM   
  mybucket_logs
```

**Results:** see below for example results. Note that this list may be incomplete as there are APIs I have not tested on my bucket: 
```
REST.DELETE.BUCKETPOLICY
REST.DELETE.METRICS
REST.GET.ACCELERATE
REST.GET.ACL
REST.GET.ANALYTICS
REST.GET.BUCKET
REST.GET.BUCKETPOLICY
REST.GET.CORS
REST.GET.ENCRYPTION
REST.GET.INVENTORY
REST.GET.LIFECYCLE
REST.GET.LOCATION
REST.GET.LOGGING_STATUS
REST.GET.METRICS
REST.GET.NOTIFICATION
REST.GET.OBJECT
REST.GET.OBJECT_LOCK_CONFIGURATION
REST.GET.POLICY_STATUS
REST.GET.PUBLIC_ACCESS_BLOCK
REST.GET.REPLICATION
REST.GET.REQUEST_PAYMENT
REST.GET.TAGGING
REST.GET.VERSIONING
REST.GET.WEBSITE
REST.HEAD.BUCKET
REST.HEAD.OBJECT
REST.OPTIONS.PREFLIGHT
REST.PUT.ANALYTICS
REST.PUT.BUCKETPOLICY
REST.PUT.INVENTORY
REST.PUT.LOGGING_STATUS
REST.PUT.METRICS
REST.PUT.OBJECT
REST.PUT.PART
REST.PUT.PUBLIC_ACCESS_BLOCK
REST.PUT.TAGGING
```

## View count of operations and success/errors on bucket in last 10 days

```sql
SELECT
  bucket
  ,operation
  ,httpstatus
  ,errorcode
  ,count(*) AS count
FROM   
  mybucket_logs
WHERE     
  parse_datetime(requestdatetime,'dd/MMM/yyyy:HH:mm:ss Z')
    >= (CURRENT_DATE - interval '10' day)
GROUP BY
  bucket
  ,operation 
  ,httpstatus
  ,errorcode
ORDER BY  
  httpstatus desc, 
  count desc
```

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
  AND parse_datetime(requestdatetime,'dd/MMM/yyyy:HH:mm:ss Z')
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
