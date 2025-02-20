
#### **CloudWatch**

AWS CloudWatch is a monitoring and observability platform that gives us greater insight into our AWS environment by monitoring applications at multiple levels. 

> *note :  A CloudWatch agent must be installed on the appropriate instance for application and system metrics to be captured.*

#### **CloudTrail**

AWS CloudTrail is used to track or monitor actions in your AWS Environment 

Essentially, any action the user takes (via the management console or AWS CLI) or service will be captured and stored.

Cloudtrail can capture logs from AWS services like S3 , IAM ....

#### **JQ**

used to transform JSON data into meaningful data we can understand and use to gain security insights. also JQ is a command line process. Ex: `jq '.[]' book_list.json`

jq <filter> <file.json>

If we wanted to view all the book titles contained within this JSON file, this would return a nicely formatted output like this:

jq '.[] | .book_title' book_list.json

jq -r '.Records[] | select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares")' cloudtrail_log.json

![[JQ Ex.png]]

if result is overwhelming we can still filter and only select significant fields :

jq -r '.Records[] | select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares") | [.eventTime, .eventName, .userIdentity.userName // "N/A",.requestParameters.bucketName // "N/A", .requestParameters.key // "N/A", .sourceIPAddress // "N/A"]' cloudtrail_log.json

**Put the result into a table :**

jq -r '["Event_Time", "Event_Name", "User_Name", "Bucket_Name", "Key", "Source_IP"],(.Records[] | select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares") | [.eventTime, .eventName, .userIdentity.userName // "N/A",.requestParameters.bucketName // "N/A", .requestParameters.key // "N/A", .sourceIPAddress // "N/A"]) | @tsv' cloudtrail_log.json | column -t

//investigating more about the user "glitch"

jq -r '["Event_Time", "Event_Source", "Event_Name", "User_Name", "Source_IP"],(.Records[] | select(.userIdentity.userName == "glitch") | [.eventTime, .eventSource, .eventName, .userIdentity.userName // "N/A", .sourceIPAddress // "N/A"]) | @tsv' cloudtrail_log.json | column -t -s $'\t'

//investigate event of Privilege given to a user 

jq '.Records[] |select(.eventSource=="iam.amazonaws.com" and .eventName== "AttachUserPolicy" )' cloudtrail_log.json