# terraform-aws-elb-logs-to-elasticsearch
Send ELB logs from S3 bucket to ElasticSearch using AWS Lambda

This directory contains terraform module to send ELB logs from S3 bucket to ElasticSearch using AWS Lambda

Particularly it creates:

1. Lambda function that does the sending
2. IAM role and policy that allows access to ES
3. S3 bucket notification that triggers the lambda function when an S3 object is created in the bucket.
4. (Only when your Lambda is deployed inside a VPC) Securitygroup for Lambda function

## Module Input Variables


| Variable Name | Example Value | Description | Default Value | Required |
| --- | --- | --- | --- |  --- |
| es_endpoint | search-es-demo-zveqnhnhjqm5flntemgmx5iuya.eu-west-1.es.amazonaws.com  | AWS ES fqdn without http:// | `None` | True |
| index |  `elblogs` | Index to create. adds a timestamp to index. Example: elblogs-2016.03.31` | `elblogs` | False |
| doctype  | `elb-access-logs` | doctype | `elb-access-logs` |  False |
| region | `eu-west-1` | AWS region | `eu-west-1` |  False |
| nodejs_version | `8.10` | Nodejs version to be used | `8.10` |  False |
| prefix | `public-` | A prefix for the resource names, this helps create multiple instances of this stack for different environments | | False |
| s3_bucket_arn | elb-logs | The arn of the s3 bucket containing the elb logs | `None` | True |
| s3_bucket_id | elb-logs | The id of the s3 bucket containing the elb logs | `None` | True |
| subnet_ids | `["subnet-1111111", "subnet-222222"]` | Subnet IDs you want to deploy the lambda in. Only fill this in if you want to deploy your Lambda function inside a VPC. | | False |

## Example

```
provider "aws" {
  region = "eu-central-1"
}

module "public_elb_logs_to_elasticsearch" {
  source        = "neillturner/elb-logs-to-elasticsearch/aws"
  version       = "0.1.0"

  prefix        = "public_es_"
  es_endpoint   = "test-es-XXXXXXX.eu-central-1.es.amazonaws.com"
  s3_bucket_arn = "arn:aws:s3:::XXXXXXX-elb-logs-eu-west-1"
  s3_bucket_id  = "XXXXXXX-elb-logs-eu-west-1"
}

module "vpc_elb_logs_to_elasticsearch" {
  source        = "neillturner/elb-logs-to-elasticsearch/aws"
  version       = "0.1.0"

  prefix        = "vpc_es_"
  es_endpoint   = "vpc-gc-demo-vpc-gloo5rzcdhyiykwdlots2hdjla.eu-central-1.es.amazonaws.com"
  s3_bucket_arn = "arn:aws:s3:::XXXXXXX-elb-logs-eu-west-1"
  s3_bucket_id  = "XXXXXXX-elb-logs-eu-west-1"
  subnet_ids    = ["subnet-d9990999"]
}
```

## Deployment Package Creation

The zip file elb-logs-to-elasticsearch.zip was build from [prithviraju/aws-elb-logs-to-elasticsearch](https://github.com/prithviraju/aws-elb-logs-to-elasticsearch)

1. On your development machine, download and install [Node.js](https://nodejs.org/en/).
2. Go to root folder of the repository and install node dependencies by running:

   ```
   npm install
   ```

   Verify that these are installed within the `node_modules` subdirectory.
4. To fix an issue with the module node-elb-log-parser causing errors

   ```
   TypeError: Cannot read property 'split' of undefined
   ```

   manually apply fix:
   [TypeError: Cannot read property 'split' of undefined #6 #7](https://github.com/toshihirock/node-elb-log-parser/pull/7)
5. Create a zip file to package the *index.js* and the `node_modules` directory

   ```
   zip -r9 elb-logs-to-elasticsearch.zip *
   ```

The zip file thus created is the Lambda Deployment Package.
