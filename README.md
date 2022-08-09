# Creating an AWS End-of-Life Warning Dashboard

This project shows how to deploy an End-of-Life (EOL) warning dashboard that users can use to stay informed and up-to-date about EOL concerns within their account. This dashboard collects configuration information from [Amazon Relational Database Service](https://aws.amazon.com/rds/) (RDS) Instances, [Amazon Elastic Compute Cloud](https://aws.amazon.com/ec2/) Amazon Machine Images (EC2 AMIs), and [AWS Lambda Runtimes](https://aws.amazon.com/lambda/). 

## Supporting Blog Posts
[End-of-Life Warning Dashboard](https://quip-amazon.com/JzDSAn5nla0Y/End-of-Life-Warning-Dashboard)

## Architecture
![](assets/EOLDashboardv5.5.png)

1. A user requests a manual refresh of the data CSV files in S3 by sending a GET request to the API.
2. The API triggers a Lambda proxy function that creates and sends a refresh event to EventBridge using the put_events (https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/events.html#EventBridge.Client.put_events) command. 
    - Alternatively, a scheduled refresh is activated within EventBridge on a daily basis.

3. A custom EventBus triggers the EventBridge rule to invoke the data capture Lambda functions.
4. Lambda functions gathering data for Lambda runtime and RDS instance access necessary data from AWS Config while the EC2 instance Lambda function retrieves data with the describe_instance (https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html#EC2.Client.describe_instances) command. Each function overwrites its respective CSV with the updated data. Additional calculations are performed such as determining the gap in major versions between in-use and most recent versions for engines and runtimes within RDS and Lambda.
5. CSV files are uploaded to an S3 bucket (designated by the user upon installation) using put_object (https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.put_object).
6. QuickSight utilizes JSON manifest files hosted in the S3 bucket to designate the CSV files as data sources and builds a SPICE (https://docs.aws.amazon.com/quicksight/latest/user/spice.html) data set for each source. Data sets refresh (https://docs.aws.amazon.com/quicksight/latest/user/refreshing-imported-data.html) manually or on schedules determined and activated by the user.
7. QuickSight loads and visualizes the data for the user and optionally sends threshold alerts (https://docs.aws.amazon.com/quicksight/latest/user/threshold-alerts.html) in the form of email notifications if at-risk RDS or Lambda instances are detected.

### AWS Services used in the solution
- [Amazon QuickSight](https://aws.amazon.com/quicksight/) 
- [AWS Lambda](https://aws.amazon.com/lambda/)
- [Amazon EventBridge](https://aws.amazon.com/eventbridge/)
- [AWS Config](https://aws.amazon.com/config/)
- [Amazon API Gateway](https://aws.amazon.com/api-gateway/)
- [Amazon Simple Storage Service](https://aws.amazon.com/s3/) 

## Prerequisites
* [An AWS account](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&client_id=signup)
* [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
* [Python 3](https://www.python.org/downloads/)
* [An AWS Identity and Access Management](http://aws.amazon.com/iam)(IAM) role with appropriate access
* [AWS QuickSight Enterprise Edition](https://aws.amazon.com/quicksight/pricing/)

