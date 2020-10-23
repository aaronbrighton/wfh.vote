# wfh.vote

Frontend/Backend source for web service running live at: [https://wfh.vote/](https://wfh.vote/)

## Requirements:

* AWS CLI
	* With IAM user created and credentials configured within your shell: [https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)
* SAM CLI
* Python 3
* An AWS Account
        - (Optionally) a domain who's DNS is managed/hosted in Route53 within the same account

## Checklist of decisions (jot answers to these down before starting):

##### 1. Do I want to use a custom domain for the frontend and/or backend votes service (optional)? If so:
- **`***CustomDomain***`**: (replace this the domain name, a higher level domain matching this needs to exist in Route53, ex: dev.wfh.vote)
- **`***CustomDomainZoneId***`**: (replace this with the Route53 zone ID for that higher level domain, ex. if we have a zone for wfh.vote, this would be it's zone id)
- **`***CustomApiDomain***`**: "
- **`***CustomApiDomainZoneId***`**: "
##### 3. Do I want to prepend the frontend site's HTML "`<title>`" tag content with an environment name?
- **`***ENVIRONMENT_NAME***`**: (replace this with the custom environment name, or leave it blank)
##### CloudFormation stack name for frontend service and backend votes service:
- **`***CloudFormationFrontendName***`**: (name you'll give to the CloudFormation frontend stack name)
- **`***CloudFormationBackendVotesName***`**: (name you'll give to the CloudFormation backend votes stack name)

An example of the above list might look like:

```
***CustomDomain***: dev.wfh.vote
***CustomDomainZoneId***: Z3P5QSUBK4POTI
***CustomApiDomain***: api.dev.wfh.vote
***CustomApiDomainZoneId***: Z3P5QSUBK4POTI
***ENVIRONMENT_NAME***: DEV
***CloudFormationFrontendName***: wfhsurveydevfrontend
***CloudFormationBackendVotesName***: wfhsurveydevbackendvotes
```

Or:

````
***ENVIRONMENT_NAME***:
***CloudFormationFrontendName***: wfhsurveydevfrontend
***CloudFormationBackendVotesName***: wfhsurveydevbackendvotes 
````


## Usage instructions:

##### 1. Download contents of this repo
##### 2. Deploy the frontend service CloudFormation stack:
- Without custom domains:
```
aws cloudformation create-stack --stack-name ***CloudFormationFrontendName*** --template-body file://frontend/cf.yml
```
- With custom domains:
```
aws cloudformation create-stack --stack-name ***CloudFormationFrontendName*** --template-body file://frontend/cf.yml --parameters ParameterKey=CustomDomain,ParameterValue=***CustomDomain*** ParameterKey=CustomDomainZoneId,ParameterValue=***CustomDomainZoneId***
```
##### 3. Periodically describe the frontend stack until it's "StackStatus" is "CREATE_COMPLETE":
```
aws cloudformation describe-stacks --stack-name ***CloudFormationFrontendName***
```
##### 4. Make note of the value of "OutputValue" key associated with the "S3Bucket" and "CloudFrontDistribution" outputs, you'll need them in step 8 and 9 respectively.
##### 5. Deploy the backend votes service CloudFormation stack:
- Without custom domains:
```
sam build --template-file services/votes/sam.yml --build-dir .aws-sam/build
cd .aws-sam/build
sam package --output-template-file deploy.yml --s3-bucket aaronbrighton-cf-resources
aws cloudformation create-stack --stack-name ***CloudFormationBackendVotesName*** --template-body file://deploy.yml -parameters ParameterKey=FrontEndCloudFormationStackName,ParameterValue=***CloudFormationFrontendName*** --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_IAM
```
- With custom domains:
```
sam build --template-file services/votes/sam.yml --build-dir .aws-sam/build
cd .aws-sam/build
sam package --output-template-file deploy.yml --s3-bucket aaronbrighton-cf-resources
aws cloudformation create-stack --stack-name ***CloudFormationBackendVotesName*** --template-body file://deploy.yml -parameters ParameterKey=FrontEndCloudFormationStackName,ParameterValue=***CloudFormationFrontendName*** ParameterKey=CustomApiDomain,ParameterValue=***CustomApiDomain*** ParameterKey=CustomApiDomainZoneId,ParameterValue=***CustomDomainApiZoneId*** --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_IAM
```
##### 6. Periodically describe the backend stack until it's "StackStatus" is "CREATE_COMPLATE":
```
aws cloudformation describe-stacks --stack-name ***CloudFormationBackendVotesName***
```
##### 7. Make note of the "OutputValue" associated with the "CloudFrontDistribution" output (this is your `***API_ENDPOINT***`), you'll need it in step 8.
##### 8. Update the frontend static resources with the API endpoint url and deploy them to the frontend service S3 bucket:
```
cd ../../frontend/src
sed -i "s/{ENVIRONMENT_NAME}/***ENVIRONMENT_NAME*** -/g" index.html
sed -i "s/{API_ENDPOINT}/***API_ENDPOINT***/g" index.html
aws s3 sync . s3://***STEP_8_VALUE***
```
##### 9. That's it, checkout your new voting site setup at the URL you noted down in step 3 under the "CloudFrontDistribution" output.