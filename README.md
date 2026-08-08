# Problem
Organizations need continous changes in order to keep up with production workloads. Traditional deployments causes downtime,service disruption and difficulty rolling back when issues are fornd. Deployment teams need safe deployment strategies that ensure
high availabilty. Without proper deployment patterns, a single faulty decision can impact thousands of users.

# Solution
Implement Lambda Deployment patterns using Function versions,aliases, and weighted traffic routing to acheive Blue-Green and
Canary deployment models. This approach leverages Lambda's built in
versioning system combined with API gateway to control traffic distribution among different versions. Ensures Zero-Downtime and instant rollback.

# Architecture Diagram
![alt text](image.png)

## Prerequisites
1.AWS account
2.AWS CLI v2
3.Estimated cost 1-5 $
Note: Ensure your Lambda execution role has CloudWatch Logs permissions for monitoring deployment health and performance metrics.

## Key components
# Lambda function with published immutable versions
# Lambda alias(prod) with weighted routing across versions
# API Gateway REST API(AWS_PROXY integration -> alias, not fixed version)
# CloudWatch alarms on Errors metric to catch bad canaries
# IAM role scoped to AWSLambdaBasicExecutionRole only

## Tech Stack
Iac --> Terraform
Compute --> AWS Lambda(Python 3.13)
API --> Amazon API Gateway(REST)
Monitoring --> Amazon CloudWatch(metrics+alarm)
IAM-->Least privilege

## Prerequisites
AWS account + credentials configured
Terraform >= 1.5
Python 3.13
Estimated cost:$ 1-5

## Project Structure
.
├── main.tf                 # Provider config + root module wiring
├── variables.tf             # Input variables (function name, weights, region, etc.)
├── outputs.tf                # API endpoint URL, alias ARN, function ARNs
├── iam.tf                   # Execution role + policy attachment
├── lambda.tf                 # Function resource, versions, alias, routing config
├── api_gateway.tf            # REST API, resource, method, integration, deployment, permission
├── cloudwatch.tf              # Error-rate alarm
├── src/
│   ├── v1/
│   │   └── lambda_function.py   # Blue environment handler
│   └── v2/
│       └── lambda_function.py   # Green environment handler
├── terraform.tfvars.example   # Sample variable values
└── README.md

## USAGE
# 1.Initialize
terraform init

# 2.Review the plan
terraform plan -out=tfplan

# 3.Deploy (Version 1 only -100% Blue)
terraform apply tfplan

# 4.Ship a canary (Version 2 at 10% traffic)
Update the Lambda source, then set canary weight variable

# terraform.tfvars
canary_weight = 0.10   # 10% of traffic to the new version
terraform plan -out=tfplan
terraform apply tfplan
Terraform publishes the new version and updates the alias's routing_config so ~10% of requests hit the new code.

# Promote to 100% (Complete Blue-Green)
Set canary_weight=0 (Or remove the routing config) so alias points fully at new version

# Rollback
Roll back instantly by re-pointing the alias's function_version back to the previous published version in lambda.tf (or via a previous_version variable) and terraform apply — no rebuild required, since versions are immutable.

# Cleanup
terraform destroy
