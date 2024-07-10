# cf-auto-tag

## What is Auto Tag?

Auto-tag monitors events coming from CloudTrail and automatically Tag resources after being created with a 'CreatedBy' tag.

## How it works?

1. A resource is created
2. An event is generated in CloudTrail
3. Event bridge receives the event
4. Event bridge triggers a lambda
5. Lambda finds the ARN of the resource created and the user that created (using the event data)
6. Lambda uses resource-groups-tagging-api to tag the resource with the CreatedBy tag

## Requirements

- CloudTrail must be enabled in the region that this stack is deployed.

## Supported Resources and Limitations

Supported:
- EC2s (RunInstances API call)
- S3 (CreateBucket)
- IAM Roles (CreateRole) - only when deployed to us-east-1
- IAM Users (CreateUser) - only when deployed to us-east-1
- Load Balancers (CreateLoadBalancer)
- RDS Instances (CreateDBInstance)
- RDS Clusters/Aurora (CreateDBCluster)
- Lambda Functions (CreateFunction20150331)
- DynamoDB Tables (CreateTable)

Not supported:
- CloudFormation (CreateStack) was removed because to update tags it requires an UpdateStack event and that also updates tags on all resources created by the CloudFormation stack, requiring policies for our lambda covering all the resources possible to be created by CloudFormation. Due to the complexity we decided to not include.
- Deploying in the Management Account (with an Organization CloudTrail) is NOT supported as it would require to for the lambda to assume role into the individual account to add the tags.

Limitations:
- Some global resources (like IAM) only work when this stack is deployed to us-east-1.
- Must be deployed in each individual AWS Accounts, Organization Trails are not supported. (See above).

Caveats:
- In theory it should work on multiple regions, but it was not tested.
- It was only tested with users from AWS Identity Centre. If not working with IAM Users, check `event['detail']['userIdentity']` in the code where it gets the user name and adjust for IAM Users (see Troubleshooting section below).

## How to use

1. Go to cloudformation and Create Stack.
2. Select `auto-tag.cf.yml`.

To test, create an S3 bucket and in a few seconds it should add a CreatedBy tag with your username. 

## Troubleshooting

1. Find the log group `/aws/lambda/AWSAutoTag-<stack_name>`
2. Browse the logs to find the error.
