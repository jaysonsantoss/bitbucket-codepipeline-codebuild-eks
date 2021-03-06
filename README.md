# Create a CI/CD pipeline using Bitbucket, AWS CodePipeline and others.

The purpose of this document is to demonstrate an example of implementing a ci/cd framework using bitbucket and services on AWS. this project can also be used as a basis for full implementation using AWS if you replace your Bitbucket with a CodeCommit, for example. initially creates this document because I couldn't easily find information on how to integrate bitbucket with AWS tools for a deployment to EKS.

This project aims to show the necessary steps to create a ci/cd structure using bitbucket, codepipeline, codebuild, ECR and EKS.


# Requirements

Prior to executing this project, it is necessary that you already have the following things:

###### Account AWS

###### Account Bitbucket

###### Cluster EKS


# Descriptions of services that will be used

**Pipeline de AWS**
  
* Fully Managed

* Built for Scale

* Integrate with AWS services

* Secure

**Bitbucket**

Bitbucket is a Git-based code hosting and collaboration tool designed for professional software engineering and project management teams. The BitBucket brand was acquired by Atlassian in 2010, which guarantees its tools full integration with other company services and even Jira and Trello workflows.

**AWS CodeBuild**

AWS CodeBuild is a fully managed continuous integration service that compiles source code, runs tests, and produces software packages that are ready to deploy. With CodeBuild, you don’t need to provision, manage, and scale your own build servers. CodeBuild scales continuously and processes multiple builds concurrently, so your builds are not left waiting in a queue. You can get started quickly by using prepackaged build environments, or you can create custom build environments that use your own build tools. With CodeBuild, you are charged by the minute for the compute resources you use.

**AWS CodePipeline**

AWS CodePipeline is a fully managed continuous delivery service that helps you automate your release pipelines for fast and reliable application and infrastructure updates. CodePipeline automates the build, test, and deploy phases of your release process every time there is a code change, based on the release model you define. This enables you to rapidly and reliably deliver features and updates. You can easily integrate AWS CodePipeline with third-party services such as GitHub or with your own custom plugin.

# Step by step

1. ## **Permission**

1.1. **Permission EKS**

In this step we will create the rules so that we can use the kubectl command in our EKS cluster.

- Go to IAM, in the left pane click on **policy**.
- In the next step click on **"create police"** and add the content below in the json option.
``` json
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
- Skip to the third step of the process and access a name for your policy, in our example the name will be "kubectlrolepolicy" responsible for a description if necessary and click on **"create policy"**

1.2. **Create role**

- Back to the main IAM panel, select the role option.
- Click on **"create role"** and select the option **"Another AWS account"**
- Enter your **account ID**. you can view your id in the upper left corner of the aws page or by executing the command below:
``` bash
# aws sts get-caller-identity

{
    "UserId": "X0XX0X0XXX0X0XXX0X0X",
    "Account": "000000000000",
    "Arn": "arn:aws:iam::000000000000:user/xxxxx.xxx"
}`
```
- click **"next"** after entering you account ID.
- In this new tab, filter by the name of your previously created policy and select it to add our role.
![image](https://user-images.githubusercontent.com/33422115/148550143-ba5e6967-c3b7-42a9-9adc-97a109fbf3ee.png)

- Add a name to your role and a description if necessary. in our example the policy name will be EKSkubectl.
- again on the IAM home screen, click on **"role"**, search for and select your role to access its summary
- Copy Role ARN (arn:aws:iam::000000000000:role/EKSKubectl)
____
1.3 **Create IAM roles via cloudformation**

CloudFormation is, above all, a service managed by AWS that helps organize the solutions created in the cloud, it is a fundamental part to replicate configurations between development, approval and production environments of customers, but it can also be used to replicate reusable solutions between different customers.

- Go to Cloudformation
- Click on **"build stack"** in the right corner of the screen(whith new features)
> step 1 (Specify templates)
- Create template in Designer
- Click **"Template"** (bottom of the screen)
- On the screen that opens, select the **"Template"** option in the lower tab of the **"Parameters"** option
- Pass the content below in the **json** option:
```yml
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
- Click the **"refresh diagram"** icon in the upper right corner of the screen (round arrow icon).we will have the following vision.
![image](https://user-images.githubusercontent.com/33422115/148555804-ff078e28-a624-4976-9b13-ccdb18493cfd.png)

- Click the Cloud with a arrow up in the top left corner to save the stack
- **Next**
- In "specify stack details" fill in the following way:
  - **Stack name** - Add stack name, in my example(ekscicdiamstack)
  - **Parameters** - Name of the role you created in the step "IAM and Setup", in our case EKSkubectl
> step 2 (specify stack details)
- we haven't changed anything, click **"next"**
> step 3 (configure stack options)
- under Capabilities, select **"i acknowledge that AWS CloudFormation might create IAM resources."**
- **Create stack**
- <a name="teste"></a> resources on the tab, copy for later use CodeBuildServiceRole and CodePipelineServiceRole:
  - **ekscicdiamstack-CodeBuildServiceRole-X0X0X00X0X0X**
  - **ekscicdiamstack-CodePipelineServiceRole-X0X0X00X0X0X**


1.4. **Edit Configmap**

- On your terminal with access to your EKS, edit your
```
# kubectl edit -n kube-system configmap/aws-auth
```
- add the content below to "mapRoles" by changing the "rolearn" to the one copied in the step of creating this role in **IAM and Setup**
```yml
- groups:
  - system:masters
    rolearn: arn:aws:iam::000000000000:role/EKSKubectl
    username: build
```
- Save file
____

2.  ## **Set Pipeline**

2.1. **Bitbucket**

In our test example we will deploy a web page with nginx. to work, our bitbucket project needs to have the following files:

  - **buildspec.yml** - source code compilation specification files
  - **Dockerfile** - Dockerfile is a text file with instructions to create our docker image
  - **hello-k8s.yml** - manifest file containing our service and deployment
  - **index.html** - nginx default page that will go up in our example


>All files described here are available in this repository..

2.2. **ECR**

The ECR service serves to store the images created in our project. we will create a repository in the steps below:

- Select the ECR service, click on **"Repositories"**.
- click in **"Create Repository"**
- In visibility settings, select **"Private"**
- add a name to your repository, in my example(aws-pipeline)
- click **create repository**

2.3 **create connection whith Bitbucket(codestar)**

- Select service Codepipeline
- Click on **"Settings"** and select **"connections"**

![codepipeline-2](https://user-images.githubusercontent.com/33422115/149011172-fec35be0-58ca-47bd-8b63-e90c0cf102e4.png)

- click **"create connection"**

![codepipeline-3](https://user-images.githubusercontent.com/33422115/149011355-df0efef3-68ce-40f3-805a-a513fb383431.png)


  - 1 - **Select a provider** = Bitbucket
  - 2 - **connection name** = enter a name for the connection
  - 3 - Click "**Connect to Bibucket**"

![create-bitbucket](https://user-images.githubusercontent.com/33422115/149011528-93acc505-3df4-45fc-9883-e3c80c18fe5b.png)

  - In the new tab, fill in as shown in the image below:
  - select **"install app"**

- ![codepipeline-6](https://user-images.githubusercontent.com/33422115/149014039-55a9d7f0-87ee-46d2-a929-08fe7aefdee2.png)

  - Select you workspace
  - 1 - click **"Grant access"**

-![CODEPIPELINE 7](https://user-images.githubusercontent.com/33422115/149014239-9a5903d9-abef-4f22-939b-5b8e05d179e8.png)


- Click **"connect"**

![CODEPIPELINE 8](https://user-images.githubusercontent.com/33422115/149015032-307a5ef8-63b4-438f-9599-00b60196dc9b.png)

- Then we will have our connection created, note your ARN to create the IAM policy to release this ARN

![CODEPIPELINE 9](https://user-images.githubusercontent.com/33422115/149015257-6f246971-aed6-465f-a75a-25750d94e54a.png)

1.2. **Create policy**

Now let's create a policy to be able to use codestar, the app required for communication between bitbucket and codepipeline

- Go to **IAM**, in the left pane click on **policies**.
- In the next step click on **"create policy"** and add the content below in the json option.
>**[ATTENTION!]**
> add your ARN replacing the above

``` json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "codestar-connections:UseConnection",
            "Resource": "arn:aws:codestar-connections:sa-central-1:XXXXXXXXXXXX:connection/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
        }
    ]
}
```

- Skip to the third step of the process and access a name for your policy, in our example the name will be "connection-permissions-bitbucket" responsible for a description if necessary and click on **"create policy"**

2.4. **CodePipeline**

- Select service **"Codepipeline"**
- Select **"Create pipeline"**

> step 1 (Choose pipeline settings)
###### Pipeline Settings

  - **pipeline name** - Enter the name of the pipeline. Once created, you cannot edit the pipeline name.
  - **service function** - Select "Existing Service Role"
  - **Role name** - select codepipeline role created in the [Cloudformation process](#teste) - Ex (ekscicdiamstack-CodePipelineServiceRole-X0X0X00X0X0X)
  - **click next**
  
  
> step 2 (add source step)
###### origin

  - **source provider** - This is where you stored the pipeline input artifacts. choose Bitbucket
  - **connection** - click "Connect Bitbucket"
  - In the new tab that will open, with the title **"Create a Connection"**, give a name to the connection that we are going to create with bitbucket and click on **"connect to Bitbucket"**
  - In the new tab that will open, titled **"Connect to Bitbucket"**, click on **"Install a new app"**. one more tab will open like the screen below:
    
  - ![2](https://user-images.githubusercontent.com/33422115/148586400-08e3b667-8a9e-4844-a2d9-e9ff4853a837.jpg)
 
  - Select your workspace and click **"Grant access"**
  - again on the screen **"Connect to Bitbucket"**, click **"to connect"**
  - again at the source, we will see the following message in a green box:

   ![green checkbox](https://user-images.githubusercontent.com/33422115/148785548-9473ba48-0428-4e53-afa2-8ccfbcf4f4b7.png)

    > ATTENTION!
    > go back to IAM and change codestar's launch policy with your **"arn:aws:codestar"** generated in that process.

   - **repository name** - Choose a repository in your Bitbucket account.
   - **branch name** - Choose a repository branch.
   - Change detection options
   - check a box "Start pipeline on source code change" if you want Automatically start the pipeline when a source code change occurs. If disabled, the pipeline will only run if started manually or on a schedule.
   - **Output artifact format**- select **"full clone"**
   - click **"Next"**
   
> Step 3 (Add build step)
###### Compilation 

  - **Build Provider** - select **"AWS Coodebuild"**
  - **Region** - Select a region for this service
  - **Project name**- click **"Create project"**, On the new screen that opens, fill in, confirm the description below:

     *project configuration*
      -  **Project name** -  project name must contain 2 to 255 characters. 
      -  **Description** - Descrição do seu projeto

      *Environment*
      - **Environm*ent image** - select **"managed image"**
      - **Operational system** - select **"Ubuntu"**
      - **Runtime(s)** - select **"Standart"**
      - **Image** - select **"AWS/codebuild/standart:5.0"**
      - **Image version** - select **"Always use the latest image for this version of  the runtime"** 
      - **Privileged** - select checkbox
      - **service function** - select **"Existing Service Role"**
      - **Role name** - select codepipeline role created in the [Cloudformation process](#teste)- Ex (ekscicdiamstack-CodeBuildServiceRole-X0X0X00X0X0X)
    
      *Buildspec*
       - **Build Specifications** - select "Use a buildspec file"
       - **everything else can be default**
       - click **"Continue to CodePipeline"**
  
      *Environment variables- optional* 
       - click **"Add environment"** 
       - add the 5 variables below changing according to your project:
         - REPOSITORY_URI= XXXXXXXXXXXX.dkr.ecr.eu-east-1.amazonaws.com/aws-pipeline-repo - (your previously created ECR repository)
         - REPOSITORY_NAME=aws-pipeline - (Name of your repository on Bitbucket)
         - REPOSITORY_BRANCH=main - (Your branch name)
         - EKS_CLUSTER_NAME=EKS-Workshop - (You Cluster EKS name)
         - EKS_KUBECTL_ROLE_ARN=arn:aws:iam::000000000000:role/EKSKubectl - (You ROLE_ARN)
  - **Build Type** - Select **"sigle build"**
  - click **"Next"**

> Step 4 (Add deployment stage)
- *Deploy*
   - click in **"Skip deploy stage"**

> Step 5 (Review)
- click **"Create Pipeline"**
- With that we will have our pipeline ready and it will start the process automatically. it is possible to directly follow the main screen of condepipeline.

_____

# Sumary

This document sought to briefly explain each process to be performed to use the resources described here to generate an automated pipeline for deploying containers in an EKS cluster on amazon, any suggestion will be welcome.

# Special thanks

Special thanks to **Renan Toesqui Magalhães** and **Valmir da Silva Junior** for his contribution to this project.
- https://github.com/renantmagalhaes
- https://github.com/valmirdasilvajr

# Reference to this documentation

To generate this documentation we use the document below as a basis:
- https://www.eksworkshop.com/intermediate/220_codepipeline/



