---
layout: post
title: Triggering AWS CodePipeline on Any ECR Image Push
author: Sukanya M
categories: [AWS]
tags: [AWS, CICD, Terraform]
date: 2025-09-15 10:15:00 +0800
math: true
mermaid: true

---

In a standard CI/CD setup using AWS CodePipeline with ECR, the source action usually monitors a specific tag, such as latest. However, in modern container workflows, you might use unique tags (like git commits or semantic versions) and want the pipeline to trigger regardless of what the tag is named.

By leveraging Amazon EventBridge, we can intercept the ECR "Push Success" event and pass the unique image digest directly to CodePipeline.

#### Prerequisites:

Before we begin, ensure you have the following set up:

- AWS Account: You'll need an active AWS account.
- AWS CLI Configured: The AWS Command Line Interface should be installed and configured with your credentials.
- Terraform: You'll need latest Terraform installed

#### The Architecture

1. Amazon ECR: A developer or CI system pushes a new image.

1. Amazon EventBridge: Detects the PUSH action and captures the image-digest.

1. Input Transformer: Formats the event data into a structure CodePipeline understands.

1. AWS CodePipeline: Starts a new execution using the specific image digest provided by the event.

------------------------------------------------------------------------------------------------------------------------------------------------------


#### Step 1: The EventBridge Rule

We define a rule that listens specifically for successful pushes to our repository.
```
resource "aws_cloudwatch_event_rule" "ecr_push_rule" {
  name        = "ecr-push-to-pipeline-trigger"
  description = "Trigger CodePipeline on any ECR Push"

  event_pattern = jsonencode({
    "source": ["aws.ecr"],
    "detail-type": ["ECR Image Action"],
    "detail": {
      "action-type": ["PUSH"],
      "result": ["SUCCESS"],
      "repository-name": ["my-application-repo"]
    }
  })
}
```

#### Step 2: Mapping the Event to the Pipeline

This is the "secret sauce." The input_transformer takes the image-digest from the ECR event and maps it to the sourceRevisions parameter of the StartPipelineExecution API call.

Note: The actionName in the template must match the name of your Source action in the Pipeline resource.

```
resource "aws_cloudwatch_event_target" "pipeline_target" {
  rule     = aws_cloudwatch_event_rule.ecr_push_rule.name
  arn      = aws_codepipeline.main.arn
  role_arn = aws_iam_role.eventbridge_pipeline_role.arn

  input_transformer {
    input_paths = {
      "revisionValue" = "$.detail.image-digest"
    }
    input_template = <<EOF
{
  "sourceRevisions": [
    {
      "actionName": "ECRSource",
      "revisionType": "IMAGE_DIGEST",
      "revisionValue": "<revisionValue>"
    }
  ]
}
EOF
  }
}
```

#### Step 3: Required Permissions

EventBridge needs permission to start your pipeline. Ensure your IAM role has the following trust relationship and policy:

```
resource "aws_iam_role_policy" "eventbridge_pipeline_policy" {
  name = "eventbridge-pipeline-trigger-policy"
  role = aws_iam_role.eventbridge_pipeline_role.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action   = "codepipeline:StartPipelineExecution"
        Effect   = "Allow"
        Resource = "*" # Restrict to your pipeline ARN in production
      }
    ]
  })
}
```

#### Important Configuration in CodePipeline

For this to work, your aws_codepipeline resource must use Pipeline Type V2. V2 supports the enhanced triggers and input overrides required for this flow.

```
resource "aws_codepipeline" "main" {
  name           = "container-deployment-pipeline"
  pipeline_type  = "V2" 
  # ... rest of the config
}
```

#### Conclusion
By moving the "trigger" logic out of CodePipeline and into EventBridge, you gain much more control. You are no longer restricted to the latest tag; your pipeline becomes truly event-driven, reacting instantly to any new image version published to your registry.
