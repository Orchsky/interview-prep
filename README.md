Thursday 02/16/2023

Interview: 
Question: How do you restrict s3 bucket access to Specific IP Addresses?

Answer: This can be done using the S3 bucket policies 

example policy:

```
{
    "Version": "2012-10-17",
    "Id": "Policy1676594375153",
    "Statement": [
        {
            "Sid": "Stmt1676594373720",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::orchsky-terraform-backend",
                "arn:aws:s3:::orchsky-terraform-backend/*"
            ],
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "44.202.136.61"
                }
            }
        }
    ]
}
```

aws s3api get-object --bucket orchsky-terraform-backend --key terraform.tfstate myfile


Question: How to ensure that users can't turn off CloudTrail?

Answer: 1) IAM policy 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1676596498071",
            "Action": [
                "cloudtrail:StopLogging"
            ],
            "Effect": "Deny",
            "Resource": "arn:aws:cloudtrail:us-east-2:062315287263:trail/cw-lambda-events"
        }
    ]
}
```

aws cloudtrail describe-trails --trail-name-list cw-lambda-events --region us-east-2
aws cloudtrail put-resource-policy --resource-arn arn:aws:cloudtrail:us-east-2:062315287263:trail/cw-lambda-events --resource-policy cloudtrail-stop-logging --region us-east-2
aws cloudtrail get-resource-policy --resource-arn arn:aws:cloudtrail:us-east-2:062315287263:trail/cw-lambda-events --region us-east-2

Answer: 2) Event driven (Lambda/CloudWatch rules)

```
import json 
import boto3 
import sys 

print("Loading function")

def lambda_handler(event, context):
    print(f'Return event in the output: {event}')
    client = boto3.client('cloudtrail')
    if event.get('detail').get('eventName') == 'StopLogging':
        response = client.start_logging(Name=event.get('detail').get('requestParameters').get('name'))

```

IAM policy 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:log-group:*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "logs:PutLogEvents",
            "Resource": "arn:aws:logs:*:*:log-group:*:*:*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "*"
        }
    ]
}
```

Event rule 

```

{
  "source": ["aws.cloudtrail"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventSource": ["cloudtrail.amazonaws.com"],
    "eventName": ["StopLogging"]
  }
}
```

Question: How do you Audit your AWS environment?

Answer: 1) AWS Trusted Adviser and 2) Scout2

What is AWS Trusted Adviser? AWS Trusted Advisor is an online tool that provides you real time guidance to help you provision your resources following AWS best practices.

Question: How do you capture real-time apache logs analysis?
Answer: Amazon Kinesis and Amazon Elasticsearch Service 

Step1: Setup ElasticSearch Cluster
Step2: Setup firehose delivery Pipeline, this will continuously insert logs to ElasticSearch Cluster
Step3: Send data to Firehose Delivery Stream
Step4: Visualize the data using Kibana

What is Amazon Kinesis

Amazon Kinesis is the streaming service provided by AWS which makes it easy to collect, 
process, and analyze real-time, streaming data so you can get timely insights and react quickly 
to new information.

What is AWS ElasticSearch Service

AWS ElasticSearch Service is a cost-effective managed service that makes it easy to deploy, 
manage, and scale open source Elasticsearch for log analytics, full-text search and more.
