# terraform-localstack-setup

This repository is intended to register an idea to bootstrap a study or QA [terraform](https://www.terraform.io/) project using [LocalStack](https://github.com/localstack/localstack) to simulate AWS.<br>

The backend is configured to store tfstate in local S3 bucket and to store tfstate lock in local DynamoDB, both running in a LocalStack docker container.<br>
<br>

## First steps
Follow the next steps to create initial configuration and test if everything is setup correctly.

<br>

### Install necessary software
To use this local setup you will need the following software available in your machine:

- docker
- docker-compose
- terraform cli
- awscli

<br>

### Bring up localstack container
Change to **localstack** directory and run:<br>
`docker-compose up -d`

<br>

### Configure AWSCLI
Create a configuration for AWS with dummy creadentials.<br>
The only important information to localstack is the region.<br>
Credentials validation is actually disabled.<br>
`aws configure`

<br>

### Create s3 state bucket and DynamoDB table
Change to **main** directory and run terraform:<br>
`terraform init` and `terraform apply`

<br>

#### Check if s3 bucket was created correctly:
`aws --endpoint=http://localhost:4566 s3 ls`

```
2021-01-25 22:05:18 terraform-state
```

<br>

#### Check if DynamoDB table was created correctly:
`aws --endpoint=http://localhost:4566 aws dynamodb list-tables`

```json
{
    "TableNames": [
        "terraformlock"
    ]
}
```

<br>

### Create a test resource, a tfstate and tfstate lock
Change to **dev** directory and run terraform:<br>
`terraform init` and `terraform apply`

<br>

#### Check if sample resource was created correctly:
`aws --endpoint=http://localhost:4566 sqs list-queues`

```json
{
    "QueueUrls": [
        "https://localhost:4566/000000000000/sample-queue"
    ]
}
```

<br>

#### Check if tfstate was stored correctly:
`aws --endpoint=http://localhost:4566 s3 ls terraform-state/dev/`

```
2021-01-25 22:17:34       1242 terraform.tfstate
```

<br>

#### Check if tfstate lock was stored correctly:
`aws --endpoint=http://localhost:4566 dynamodb scan --table-name terraformlock --output json`

```json
{
    "Items": [
        {
            "Digest": {
                "S": "6295c01f8c53c7a74bfad08f566d20c4"
            },
            "LockID": {
                "S": "terraform-state/dev/terraform.tfstate-md5"
            }
        }
    ],
    "Count": 1,
    "ScannedCount": 1,
    "ConsumedCapacity": null
}
```

<br>

## Extra hints
In order to not get annoyed on having to always provide `--endpoint=http://localhost:4566` in your study machine, you can do the following:
- Create an alias that will always point to the local enpoint as:<br>
    `alias awslocal='aws --endpoint-url http://localhost:4566'`.
- Put the alias in your `.bashrc` or `.zshrc` to have it permanent in your terminal session.
- If you create the alias named as "aws" you can also make use of awscli autocomplete with your localstack endpoint.