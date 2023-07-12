# dynamodb-streaming-datalake
This is a quick demonstration of capturing DynamoDB data change with Amazon Kinesis and Amazon Kinesis Data Firehose into a data lake on Amazon S3. This reference architecture is built with AWS CDK using Typescript and all of the resources are **in the same AWS account**.
## Content Table
* [Reference Architecture](#reference-architecture)  
* [Steps for quick start with AWS CDK](#steps-for-quick-start-with-aws-cdk)  
* [Test DDB CDC Progress](#test-ddb-cdc-progress)
* [Notes](#notes)  
## Reference Architecture
![reference architecture](images/ddb%20cdc%20into%20data%20lake.png)

## Steps for quick start with AWS CDK
The following commands are tested on MacBook Pro.
```bash
# Prerequisites
brew install node
node --version
brew install npm
npm --version
## I personally used to use yarn to manipulate packages, so the following installation, i.e., yarn, is optional. For the installation of CDK itself and related packages, npm and yarn all all available. I will present the yarn commands here.
brew install yarn
yarn --version

# Install AWS CDK
yarn global add aws-cdk
cdk version
# Install CDK packages
yarn install
# Upgrade all of the related packages
yarn upgrade

# Deploy with AWS CDK
export AWS_PROFILE_NAME="YOUR PROFILE NAME"
cdk synth --profile ${AWS_PROFILE_NAME}
cdk diff --profile ${AWS_PROFILE_NAME}
cdk deploy --profile ${AWS_PROFILE_NAME}
```

## Test DDB CDC Progress
Once you deploy the CDK stack, you can test the DDB CDC progress with the following commands.  
If you have multiple AWS credentials on your development machine, you can use the `--profile` option to specify which credentials to use. Otherwise, just ignore the `--profile` option.
```bash
export AWS_PROFILE_NAME="YOUR PROFILE NAME"
aws dynamodb put-item --table-name "example-ddb-table" \
	--item '{ "id": {"S": "864732"}, "name": {"S": "Adam"} , "Designation": {"S": "Architect"} }' \
	--return-consumed-capacity TOTAL \
	--profile ${AWS_PROFILE_NAME}

aws dynamodb put-item --table-name "example-ddb-table" \
	--item '{ "id": {"S": "864732"}, "name": {"S": "Adam"} , "Designation": {"S": "Sr. Architect"} }' \
	--return-consumed-capacity TOTAL \
	--profile ${AWS_PROFILE_NAME}

aws dynamodb put-item --table-name "example-ddb-table" \
	--item '{ "id": {"S": "864732"}, "name": {"S": "Adam"} , "Designation": {"S": "Developer Advocate"} }' \
	--return-consumed-capacity TOTAL \
	--profile ${AWS_PROFILE_NAME}
```
If your deployment is successful, you should be able to see similar result as the following diagram.
![reference result](images/ddb%20cdc%20result.png)

## Notes
1. In this quick demo, the data encryption at rest for Amazon Kinesis Data Stream, Amazon Kinesis Firehose and Amazon S3 are all using the same AWS KMS key. It is not necessary to use the same key for all the resources.
2. In the cross-account scenarios, the IAM role of the Kinesis Firehose delivery stream, the KMS policy of the AWS KMS key used by the data lake bucket on Amazon S3, and the bucket policy of the data lake bucket need to be taken care specifically.
3. Consumer requirements
   * Producer's account ID
   * Producer's Firehose IAM role name  
4. Producer requirements
    * Consumer's account ID
    * Consumer's datalake bucket name
    * Consumer's KMS key ARN of the datalake bucket
5. Remember to execute `cdk bootstrap` for the first time to deploy the CDK toolkit stack into your AWS account.  
```bash
# single account
AWS_PROFILE_NAME="YOUR PROFILE NAME"
ACCOUNT_ID="123456789012"
AWS_REGION="ap-northeast-1"
cdk bootstrap --profile ${AWS_PROFILE_NAME} --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess aws://${ACCOUNT_ID}/${AWS_REGION}

# cross-account, for consumer and producer
AWS_PROFILE_NAME="Consumer's Profile Name"
TRUSTED_ACCOUNT_ID="Producer's Account ID"
ACCOUNT_ID="Consumer's Account ID"
AWS_REGION="Consumer's Deployment Region"
cdk bootstrap --trust ${TRUSTED_ACCOUNT_ID} --profile ${AWS_PROFILE_NAME} --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess aws://${ACCOUNT_ID}/${AWS_REGION}
```