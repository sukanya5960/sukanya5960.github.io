---
layout: post
title: Infrastructure as Code Using AWS CDK
author: Sukanya M
categories: [AWS]
tags: [AWS, Python, Programming]
date: 2024-09-21 10:15:00 +0800
math: true
mermaid: true

---

Let's dive into how to programmatically create an ECR repository using the AWS Cloud Development Kit (CDK) with Python. CDK allows us to define our cloud infrastructure using familiar programming languages, making infrastructure as code (IaC) more accessible and maintainable.

#### Prerequisites:

Before we begin, ensure you have the following set up:

- AWS Account: You'll need an active AWS account.
- AWS CLI Configured: The AWS Command Line Interface should be installed and configured with your credentials.
- Python 3.6 or later: CDK Python requires a compatible Python version.
- Node.js and npm: CDK CLI relies on Node.js and npm.
- AWS CDK Toolkit Installed: If you haven't already, install the CDK CLI globally:
```
npm install -g aws-cdk
```

First, let's create a new CDK project in Python. Open your terminal and navigate to your desired project directory. Then, run:

```
cdk init app --language python
```

Activate the virtual environment created by CDK:

```
source .venv/bin/activate
```

Now, navigate to the project's core file, typically located under the project's root directory with a name similar to <your_project_name>_stack.py. We'll be adding our ECR repository definition here.

#### Defining the ECR Repository:

Import the necessary CDK modules and the aws_ecr module:
```
from aws_cdk import core as cdk
from aws_cdk import aws_ecr as ecr
```

Inside your stack class (which inherits from cdk.Stack), you can now define your ECR repository. Here's a basic example:

```
class MyEcrStack(cdk.Stack):

    def __init__(self, scope: cdk.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        repository = ecr.Repository(
            self,
            "MyWebAppRepo",
            repository_name="my-web-app-repo"  # Optional: Specify a custom repository name
        )

        cdk.CfnOutput(self, "RepositoryUri", value=repository.repository_uri)
```


In this code:

- We import the ecr module from aws_cdk.
- We create an instance of ecr.Repository within our stack.
- The first argument (self) refers to the current stack.
- The second argument ("MyWebAppRepo") is a logical ID for this resource within your CDK stack. It needs to be unique within the stack.
- The repository_name parameter (optional) allows you to specify a custom name for your ECR repository in AWS. If you don't provide it, CDK will generate a unique name.
- cdk.CfnOutput is used to output the URI of the newly created repository after deployment, which is essential for pushing your Docker images.


#### Deploying Your CDK Stack:

Save your changes to the stack file. Now, in your terminal, navigate to the root of your CDK project and run the following commands:

1. Synthesize the CloudFormation template:
```
cdk synth
```
This command will generate the CloudFormation template based on your Python code.

1. Deploy the CDK stack to your AWS account:
```
cdk deploy
```
CDK will prompt you for confirmation before creating the resources in your AWS account (in the AWS region you have configured). Type y to proceed.