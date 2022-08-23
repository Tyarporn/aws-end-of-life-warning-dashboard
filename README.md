# Creating an AWS End-of-Life Warning Dashboard

This project shows how to deploy an End-of-Life (EOL) warning dashboard that users can use to stay informed and up-to-date about EOL concerns within their account. This dashboard collects configuration information from [Amazon Relational Database Service](https://aws.amazon.com/rds/) (RDS) Instances, [Amazon Elastic Compute Cloud](https://aws.amazon.com/ec2/) (EC2) Instances, and [AWS Lambda runtimes](https://aws.amazon.com/lambda/). 

## Supporting Blog Posts
[End-of-Life Warning Dashboard](https://quip-amazon.com/JzDSAn5nla0Y/End-of-Life-Warning-Dashboard)

## Architecture
![](assets/EOLDashboardv6.1.png)

1. A) The user sends a POST method request to *API Gateway Manual Trigger*. The method request sends a user-generated event to EventBridge.
   
   B) The *Scheduled Rule* sends events to EventBridge on a daily basis.
    
2. After EventBridge receives an event, the corresponding rule(s) invoke their target Lambda functions to collect configuration details. *Lambda runtime* information is gathered using [list_functions](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lambda.html#Lambda.Client.list_functions), *RDS instance* information is collected with [describe_db_instances,](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-instances.html) and *EC2 instance* data is gathered with [describe_instance](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html#EC2.Client.describe_instances).
3. Each function writes the configuration details to a CSV file which is uploaded to an S3 bucket (designated by the user upon installation) using [put_object](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.put_object). Each upload will overwrite the previous version of the given CSV. 
4. QuickSight utilizes JSON manifest files hosted in the S3 bucket to designate the CSV files as data sources. A data set is built from each source. Data sets [refresh](https://docs.aws.amazon.com/quicksight/latest/user/refreshing-imported-data.html) with the latest CSV updates manually or on schedules determined and activated by the user.
5. QuickSight loads and visualizes the data for the user and optionally sends [threshold alerts](https://docs.aws.amazon.com/quicksight/latest/user/threshold-alerts.html) in the form of email notifications if at-risk RDS or Lambda instances are detected.

For more information on executing the aforementioned resource functions within Lambda, see the documentation for the AWS SDK for Python [(Boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html).




### AWS Services used in the solution
- [Amazon QuickSight](https://aws.amazon.com/quicksight/) 
- [AWS Lambda](https://aws.amazon.com/lambda/)
- [Amazon EventBridge](https://aws.amazon.com/eventbridge/)
- [Amazon API Gateway](https://aws.amazon.com/api-gateway/)
- [Amazon Simple Storage Service](https://aws.amazon.com/s3/) 

## Prerequisites
* [An AWS account](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&client_id=signup)
* [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) (Users can alternatively deploy with the [AWS Cloud9 IDE](https://aws.amazon.com/cloud9/) terminal or the AWS CloudShell Terminal)
* [AWS Command Line Interface](https://aws.amazon.com/cli/) (AWS CLI)
* [An AWS Identity and Access Management](http://aws.amazon.com/iam)(IAM) role with appropriate access
* [AWS QuickSight Enterprise Edition](https://aws.amazon.com/quicksight/pricing/)

## Project Structure
```
aws-end-of-life-warning-dashboard/
├── assets - This has the image files used for the README.md
├── manifests - This has the manifest JSON files that are needed for QuickSight. These are the files you need to change before putting into S3  
├── source
    ├── dashboard-ec2-function.zip
    ├── dashboard-init-function.zip
    ├── dashboard-lambda-function.zip
    ├── dashboard-rds-function.zip
└── template.yaml - A template that defines the application's AWS resources
```

## Solution Deployment Walkthrough
At a high-level, here are the steps to get the solution running:
1. Deploy the solution using any IDE (This guide uses Visual Studio Code).
2. Deploy the solution with the SAM CLI if step 1 is not used.
3. Test the solution.

Detailed steps are provided below:

### 1. Download the solution using Visual Studio Code and AWS Toolkit
Download the code from the Github [repository](https://github.com/Tyarporn/aws-end-of-life-warning-dashboard).
```
git clone https://github.com/Tyarporn/aws-end-of-life-warning-dashboard
```
You can follow the steps [here](https://docs.microsoft.com/en-us/azure/developer/javascript/how-to/with-visual-studio-code/clone-github-repository?tabs=create-repo-command-palette%2Cinitialize-repo-activity-bar%2Ccreate-branch-command-palette%2Ccommit-changes-command-palette%2Cpush-command-palette) to clone the project from Github
You should be able to see the project as shown here, please take a moment to review the project files:
    
![img](assets/filesss.png)
    
If you have not already, please set up your AWS crendentials for the [AWS Toolkit](https://aws.amazon.com/visualstudiocode/). Also, ensure that the [AWS SAM CLI](https://aws.amazon.com/serverless/sam/) is installed.
    
### 2. Build the solution using AWS SAM CLI
Refer to the [Supporting Blog](https://quip-amazon.com/JzDSAn5nla0Y/End-of-Life-Warning-Dashboard) post for instructions on how to use AWS SAM CLI
### 3. Use the solution
Refer to the [Supporting Blog](https://quip-amazon.com/JzDSAn5nla0Y/End-of-Life-Warning-Dashboard) post for instructions on how to use the dashboard
    
## Cleanup
To delete the application stack using the AWS CLI you can run the following command (replace stack-name with your actual stack name).
```
aws cloudformation delete-stack --stack-name <stack-name>
```
Alternatively, you can also delete the CloudFormation stack in VS Code by right clicking the project in the AWS Explorer pane as shown below:
![img](assets/samdeletess.png)
    
