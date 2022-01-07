# bitbucket-codepipeline-codebuild-eks
ci/cd project with the tools mentioned in the title

This project aims to show the necessary steps to create a ci/cd structure using bitbucket, codepipeline, codebuild, ECS and EKS.


# Requirements
Prior to executing this project, it is necessary that you already have the following things:

Account AWS

Account Bitbucket

Cluster EKS



# Descriptions of services that will be used

**Pipeline de AWS**

Fully Managed

Built for Scale

Integrate with AWS services

Secure

**Bitbucket**

Bitbucket is a Git-based code hosting and collaboration tool designed for professional software engineering and project management teams. The BitBucket brand was acquired by Atlassian in 2010, which guarantees its tools full integration with other company services and even Jira and Trello workflows.

**AWS CodeBuild**

AWS CodeBuild is a fully managed continuous integration service that compiles source code, runs tests, and produces software packages that are ready to deploy. With CodeBuild, you don’t need to provision, manage, and scale your own build servers. CodeBuild scales continuously and processes multiple builds concurrently, so your builds are not left waiting in a queue. You can get started quickly by using prepackaged build environments, or you can create custom build environments that use your own build tools. With CodeBuild, you are charged by the minute for the compute resources you use.

**AWS CodePipeline**

AWS CodePipeline is a fully managed continuous delivery service that helps you automate your release pipelines for fast and reliable application and infrastructure updates. CodePipeline automates the build, test, and deploy phases of your release process every time there is a code change, based on the release model you define. This enables you to rapidly and reliably deliver features and updates. You can easily integrate AWS CodePipeline with third-party services such as GitHub or with your own custom plugin.

# Step by step

**IAM and Setup**

In this step we will create the rules so that we can use the kubectl command in our EKS cluster.

- Go to IAM, in the left pane click on policy.
- In the next step click on "create police" and add the content below in the json option.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "eks:Describe*",
            "Resource": "*"
        }
    ]
}
```
- Skip to the third step of the process and access a name for your policy, in our example the name will be "kubectlrolepolicy" responsible for a description if necessary and click on "create policy"

- Back to the main IAM panel, select the role option.
- Click on "create role" and select the option "Another AWS account"
- Enter your account ID. you can view your id in the upper left corner of the aws page or by executing the command below:
```
# aws sts get-caller-identity

{
    "UserId": "X0XX0X0XXX0X0XXX0X0X",
    "Account": "000000000000",
    "Arn": "arn:aws:iam::000000000000:user/xxxxx.xxx"
}`
```
- click next after entering you account ID.
- In this new tab, filter by the name of your previously created policy and select it to add our role.
![image](https://user-images.githubusercontent.com/33422115/148550143-ba5e6967-c3b7-42a9-9adc-97a109fbf3ee.png)
- Add a name to your role and a description if necessary. in our example the policy name will be EKSkubectl.
- again on the IAM home screen, click on role, search for and select your role to access its summary
- Copy Role ARN (arn:aws:iam::000000000000:role/EKSKubectl)

**Create IAM roles via cloudformation**

CloudFormation is, above all, a service managed by AWS that helps organize the solutions created in the cloud, it is a fundamental part to replicate configurations between development, approval and production environments of customers, but it can also be used to replicate reusable solutions between different customers.

- Go to Cloudformation
- Click on build stack in the right corner of the screen(whith new features)
> step 1 (Specify templates)
- Create template in Designer
- Click Template (bottom of the screen)
- On the screen that opens, select the "Template" option in the lower tab of the "Parameters" option
- Pass the content below in the json option:
```
---
AWSTemplateFormatVersion: 2010-09-09


Parameters:

  KubectlRoleName:
    Type: String
    Default: EksWorkshopCodeBuildKubectlRole
    Description: IAM role used by kubectl to interact with EKS cluster
    MinLength: 3
    MaxLength: 100
    ConstraintDescription: You must enter a kubectl IAM role


Resources:


  CodePipelineServiceRole:
    Type: AWS::IAM::Role
    Properties:
      Path: /
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              Service: codepipeline.amazonaws.com
            Action: sts:AssumeRole
      Policies:
        - PolicyName: codepipeline-access
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Resource: "*"
                Effect: Allow
                Action:
                  - codebuild:StartBuild
                  - codebuild:BatchGetBuilds
                  - codecommit:GetBranch
                  - codecommit:GetCommit
                  - codecommit:UploadArchive
                  - codecommit:GetUploadArchiveStatus
                  - codecommit:CancelUploadArchive
                  - iam:PassRole
              - Resource: "*"
                Effect: Allow
                Action:
                  - s3:PutObject
                  - s3:GetObject
                  - s3:GetObjectVersion
                  - s3:GetBucketVersioning


  CodeBuildServiceRole:
    Type: AWS::IAM::Role
    Properties:
      Path: /
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              Service: codebuild.amazonaws.com
            Action: sts:AssumeRole
      Policies:
        - PolicyName: root
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Resource: !Sub arn:aws:iam::${AWS::AccountId}:role/${KubectlRoleName}
                Effect: Allow
                Action:
                  - sts:AssumeRole
              - Resource: '*'
                Effect: Allow
                Action:
                  - eks:Describe*
              - Resource: '*'
                Effect: Allow
                Action:
                  - logs:CreateLogGroup
                  - logs:CreateLogStream
                  - logs:PutLogEvents
              - Resource: '*'
                Effect: Allow
                Action:
                  - ecr:GetAuthorizationToken
              - Resource: '*'
                Effect: Allow
                Action:
                  - ec2:CreateNetworkInterface
                  - ec2:DescribeDhcpOptions
                  - ec2:DescribeNetworkInterfaces
                  - ec2:DeleteNetworkInterface
                  - ec2:DescribeSubnets
                  - ec2:DescribeSecurityGroups
                  - ec2:DescribeVpcs
                  - ec2:CreateNetworkInterfacePermission
              - Resource: "*"
                Effect: Allow
                Action:
                  - s3:GetObject
                  - s3:PutObject
                  - s3:GetObjectVersion
              - Resource: '*'
                Effect: Allow
                Action:
                  - ecr:GetDownloadUrlForLayer
                  - ecr:BatchGetImage
                  - ecr:BatchCheckLayerAvailability
                  - ecr:PutImage
                  - ecr:InitiateLayerUpload
                  - ecr:UploadLayerPart
                  - ecr:CompleteLayerUpload

```
- Click the "refresh diagram" icon in the upper right corner of the screen (round arrow icon).we will have the following vision.
![image](https://user-images.githubusercontent.com/33422115/148555804-ff078e28-a624-4976-9b13-ccdb18493cfd.png)
- Click the Cloud with a arrow up in the top left corner to save the stack
- Next
- In "specify stack details" fill in the following way:
  - Stack name = Add stack name, in my example(ekscicdiamstack)
  - Parameters = Name of the role you created in the step "IAM and Setup", in our case EKSkubectl
> step 2 (specify stack details)
- we haven't changed anything, click next
> step 3 (configure stack options)
- under Capabilities, select "i acknowledge that AWS CloudFormation might create IAM resources."
- Create stack
- resources on the tab, copy for later use CodeBuildServiceRole and CodePipelineServiceRole:
  - ekscicdiamstack-CodeBuildServiceRole-X0X0X00X0X0X
  - ekscicdiamstack-CodePipelineServiceRole-X0X0X00X0X0X

**Edit Configmap**

- On your terminal with access to your EKS, edit your
```
# kubectl edit -n kube-system configmap/aws-auth
```
- add the content below to "mapRoles" by changing the "rolearn" to the one copied in the step of creating this role in **IAM and Setup**
```
- groups:
  - system:masters
    rolearn: arn:aws:iam::000000000000:role/EKSKubectl
    username: build
```
- Save file

# Set Pipeline

**Bitbucket**

In our test example we will deploy a web page with nginx. to work, our bitbucket project needs to have the following files:

  - buildspec.yml - source code compilation specification files
  - Dockerfile - Dockerfile is a text file with instructions to create our docker image
  - hello-k8s.yml - manifest file containing our service and deployment
  - index.html - nginx default page that will go up in our example

**ECR**

The ECR service serves to store the images created in our project. we will create a repository in the steps below:

- Select the ECR service, click on "Repositories".
- click in "Create Repository"
- In visibility settings, select "Private"
- add a name to your repository, in my example(aws-pipeline)
- click create repository

**CodePipeline**









