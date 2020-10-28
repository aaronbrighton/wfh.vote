# wfh.vote

Frontend/Backend source for web service running live at: [https://wfh.vote/](https://wfh.vote/)

## Prequisites

* An AWS Account
	* (Optionally) a domain who's DNS is managed/hosted in Route53 within the same account
* AWS CLI
	* With IAM user (Admin Perms.) created and credentials configured within your shell: [https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)
	* IAM Git credentials generated and stored: [https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html#setting-up-gc-iam](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html#setting-up-gc-iam)
* SAM CLI
* Python 3
* Git (of course)

## Getting started (customization)

Jot down answers to the following questions:

##### 1. What name should I use for the CodePipeline pipeline? (**`***CodePipelineName***`**)
##### 2. What name should I use for the CodeRepo repository? (**`***CodeRepoName***`**)
##### 3. (Optional) Do I want to use a custom domain for the frontend? (**`***CustomDomain***`**)
- What is the Route53 Zone ID where I can create this DNS record? (**`***CustomDomainZoneId***`**)
##### 4. (Optional) Do I want to use a custom domain for the backend votes API? (**`***CustomApiDomain***`**)
- What is the Route53 Zone ID where I can create this DNS record? (**`***CustomApiDomainZoneId***`**)
##### 5. (Optional) Do I want to add a custom environment name to the start of the frontend `<title>` tag? (**`***EnvironmentName***`**)


## Usage instructions:

##### 1. Clone the repo
```
git clone https://github.com/aaronbrighton/wfh.vote.git
```
##### 2. Deploy the pipeline stack (fill in the `***blanks***`)
```
aws cloudformation create-stack --stack-name ***CodePipelineName*** \
--template-body file://pipeline-cf.yml --parameters \
ParameterKey=CodePipelineName,ParameterValue=***CodePipelineName*** \
ParameterKey=CodeRepoName,ParameterValue=***CodeRepoName*** \
ParameterKey=CustomDomain,ParameterValue=***CustomDomain*** \
ParameterKey=CustomDomainZoneId,ParameterValue=***CustomDomainZoneId*** \
ParameterKey=CustomApiDomain,ParameterValue=***CustomApiDomain*** \
ParameterKey=CustomApiDomainZoneId,ParameterValue=***CustomApiDomainZoneId*** \
ParameterKey=EnvironmentName,ParameterValue=***EnvironmentName***
```
##### 3. Periodically describe the pipeline stack until it's "StackStatus" is "CREATE_COMPLETE"
```
aws cloudformation describe-stacks --stack-name ***CodePipelineName***
```
##### 4. Make note of the "CodeCommitHTTPCloneUrl" output value once "StackStatus" is "CREATE_COMPLETE"
##### 5. Update your git config to use the new CodeCommit repository
```
git init
git remote add origin ***CodeCommitHTTPCloneUrl***
git add *
git commit -m "Initial commit"
git push --set-upstream origin master
```
##### 5. Periodically describe the pipeline until the stage "DeployFrontendStaticAssets" has a status of "Succeeded"
```
aws codepipeline get-pipeline-state --name ***CodePipelineName***
```
##### 6. Describe the frontend stack to get the URL for your live depoyment

```
aws cloudformation describe-stacks --stack-name CF-***CodePipelineName***-Frontend
```
##### 7. Make note of the "CloudFrontDistribution" output value, this is the URL to your live site!