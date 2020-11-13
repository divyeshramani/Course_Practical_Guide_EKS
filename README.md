# A Practical Guide to EKS

In this repository you will find all the assets required for the course `A Practical Guide to Amazon EKS`, by A Cloud Guru.


## Bookstore application

This solution has been built for for explaining all the concepts in this course. It is complete enough for covering a real case of microservices running on EKS and integrating with other AWS Services.

> You can find in [here](_docs/api.md) the documentation of the APIs.


## Run on Local

To Run application on local with pointing to DynamoDB in your AWS account to store data, run following. 
```
export AWS_DEFAULT_REGION=us-east-1
export AWS_ACCESS_KEY_ID=<YOUR_AWS_ACCESS_KEY_ID>
export AWS_SECRET_ACCESS_KEY=<YOUR_AWS_SECRET_ACCESS_KEY>

# Create DynamoDB tables
aws cloudformation create-stack --stack-name dev-client-api-table --template-body file://$PWD/clients-api/infra/cloudformation/dynamodb-table.json

aws cloudformation create-stack --stack-name dev-inventory-api-table --template-body file://$PWD/inventory-api/infra/cloudformation/dynamodb-table.json

aws cloudformation create-stack --stack-name dev-renting-api-table --template-body file://$PWD/renting-api/infra/cloudformation/dynamodb-table.json

aws cloudformation create-stack --stack-name dev-resource-api-table --template-body file://$PWD/resource-api/infra/cloudformation/dynamodb-table.json


# Start Docker Containers
docker-compose up -d
```
> Open Browser and hit url http://localhost


