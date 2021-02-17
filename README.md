# wfh.vote

Simple serverless voting service for employees to anonymously vote on their work from home preferences in the context of their company.

See [wfh.vote](https://wfh.vote/) for a live example.


## Features

- Direct deployment via [AWS CloudFormation](https://aws.amazon.com/cloudformation/)
- Pipeline deployment via [AWS CodePipeline](https://aws.amazon.com/codepipeline/)
- Custom domain name support via [Amazon Route 53](https://aws.amazon.com/route53/) + automatic SSL via [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/)


## Architecture

![AWS Architecture Diagram](docs/wfh.vote.drawio.png)

*Made using [draw.io](https://app.diagrams.net/), source file: [wfh.vote.drawio](docs/wfh.vote.drawio)*


## Prequisites

* [Git](https://git-scm.com/) (of course)
* [AWS Account](https://portal.aws.amazon.com/billing/signup#/), Optionally: [Route53 Public Hosted Zone](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/CreatingHostedZone.html)
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config), Optionally: [Git Credentials](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html#setting-up-gc-iam)
* [Python 3](https://www.python.org/downloads/)
* Optionally: [SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)


## Quick Deployment: CloudFormation

```
# Clone the repo

git clone https://github.com/aaronbrighton/wfh.vote.git
cd wfh.vote

# Deploy the FrontendStack

aws cloudformation create-stack --stack-name WFHVoteFrontendStack --template-body file://frontend/cf.yml

# Build & Deploy the BackendStack

sam build --template-file services/votes/sam.yml --build-dir .aws-sam/build
cd .aws-sam/build
sam deploy --template-file template.yaml --stack-name WFHVoteBackendStack --parameter-overrides ParameterKey=FrontendCloudFormationStackName,ParameterValue=WFHVoteFrontendStack --guided
cd ../../frontend/src/

# Deploy FrontendStack static assets
# WAIT FOR CF STACKS TO FINISH DEPLOYING FIRST

FRONTEND_S3_BUCKET=`aws cloudformation describe-stacks --stack-name WFHVoteFrontendStack --query "Stacks[0].Outputs[?OutputKey=='S3Bucket'].OutputValue" --output text`
API_ENDPOINT=`aws cloudformation describe-stacks --stack-name WFHVoteBackendStack --query "Stacks[0].Outputs[?OutputKey=='CloudFrontDistribution'].OutputValue" --output text`
sed -i "s/{ENVIRONMENT_NAME}//g" index.html # If this fails, find/replace manually
sed -i "s;{API_ENDPOINT};$API_ENDPOINT;g" index.html
aws s3 sync --cache-control no-store --delete . s3://$FRONTEND_S3_BUCKET

# Return the Frontend URL
echo `aws cloudformation describe-stacks --stack-name WFHVoteFrontendStack --query "Stacks[0].Outputs[?OutputKey=='CloudFrontDistribution'].OutputValue" --output text`
```


## Complex Deployment: Pipeline Deployment

This setup requires [Git Credentials](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html#setting-up-gc-iam) to be setup to allow pushing to AWS CodeCommit repositories.

```
# Clone the repo

git clone https://github.com/aaronbrighton/wfh.vote.git
cd wfh.vote

# Deploy the PipelineStack

aws cloudformation create-stack --stack-name WFHVotePipelineStack \
--template-body file://pipeline-cf.yml --capabilities CAPABILITY_IAM --parameters \
ParameterKey=CodePipelineName,ParameterValue=WFHVotePipeline \
ParameterKey=CodeRepoName,ParameterValue=WFHVoteRepo

# Push code to new CodeCommit repository
# WAIT FOR CF PipelineStack TO FINISH DEPLOYING FIRST

CODE_COMMIT_HTTP_URL=`aws cloudformation describe-stacks --stack-name WFHVotePipelineStack --query "Stacks[0].Outputs[?OutputKey=='CodeCommitHTTPCloneUrl'].OutputValue" --output text`
rm -rf .git
git init
git remote add origin $CODE_COMMIT_HTTP_URL
git add *
git commit -m "Initial commit"
git push --set-upstream origin master

# Return the Frontend URL
# WAIT FOR CodePipeline TO FINISH DEPLOYING FIRST

echo `aws cloudformation describe-stacks --stack-name CF-WFHVotePipeline-Frontend --query "Stacks[0].Outputs[?OutputKey=='CloudFrontDistribution'].OutputValue" --output text`
```

## Additional Options

### Custom Domain Name

If you have an existing domain name in Route53, you can use the following stack parameters to customize the Frontend and Backend CloudFront domain names.  These are cutouts from the above commands with the additional parameters, reference only.

```
# Quick Deployment

aws cloudformation create-stack --stack-name WFHVoteFrontendStack --template-body file://frontend/cf.yml --parameters \
ParameterKey=CustomDomain,ParameterValue=wfh.vote \
ParameterKey=CustomDomainZoneId,ParameterValue=Z07514403HBWCQT041GV0 \

sam deploy --template-file template.yaml --stack-name WFHVoteBackendStack --guided --parameter-overrides \ ParameterKey=FrontendCloudFormationStackName,ParameterValue=WFHVoteFrontendStack \
ParameterKey=CustomApiDomain,ParameterValue=api.wfh.vote \
ParameterKey=CustomApiDomainZoneId,ParameterValue=Z07514403HBWCQT041GV0 \


# Pipeline Deployment

aws cloudformation create-stack --stack-name WFHVotePipelineStack \
--template-body file://pipeline-cf.yml --capabilities CAPABILITY_IAM --parameters \
ParameterKey=CodePipelineName,ParameterValue=WFHVotePipeline \
ParameterKey=CodeRepoName,ParameterValue=WFHVoteRepo \
ParameterKey=CustomDomain,ParameterValue=wfh.vote \
ParameterKey=CustomDomainZoneId,ParameterValue=Z07514403HBWCQT041GV0 \
ParameterKey=CustomApiDomain,ParameterValue=api.wfh.vote \
ParameterKey=CustomApiDomainZoneId,ParameterValue=Z07514403HBWCQT041GV0 \
```