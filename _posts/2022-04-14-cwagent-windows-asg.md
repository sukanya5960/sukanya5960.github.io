---
layout: post
title: AWS: CloudwatchAgent Setup Windows with Autoscaling
author: Sukanya M
categories: [AWS]
tags: [AWS, devops, ]
---

## AWS: CloudwatchAgent Setup Windows with Autoscaling

In order to monitor memory and disk usage, you need to install Cloudwatch agent in the EC2 instance.

Create IAM role > CloudWatchAgentServerPolicy > and attach it to the instance 

Add [CloudWatchAgentAdminPolicy](https://console.aws.amazon.com/iam/home?region=us-east-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FCloudWatchAgentAdminPolicy) and [AmazonEC2RoleforSSM](https://console.aws.amazon.com/iam/home?region=us-east-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2Fservice-role%2FAmazonEC2RoleforSSM)

Access Windows server >> Powershell 

Download the installation file
```sh
 Invoke-WebRequest -Uri "https://s3.amazonaws.com/amazoncloudwatch-agent/windows/amd64/latest/AmazonCloudWatchAgent.zip" -OutFile "C:\AwsCloudWatchAgent.zip"
 ```
Unzip the file from C drive

Double click the exe file to install it.

Goto the installation folder:

```sh
 C:\Program Files\Amazon\AmazonCloudWatchAgent
```
Open the file config.json (if there is no such file, create one)

Copy the following to the config.json file

```sh
{
	"metrics": {
		"aggregation_dimensions": [
			[
				"AutoScalingGroupName"
			]
		],
		"append_dimensions": {
			"AutoScalingGroupName": "${aws:AutoScalingGroupName}",
			"ImageId": "${aws:ImageId}",
			"InstanceId": "${aws:InstanceId}",
			"InstanceType": "${aws:InstanceType}"
		},
		"metrics_collected": {
			"LogicalDisk": {
				"measurement": [
					"% Free Space"
				],
				"metrics_collection_interval": 60,
				"resources": [
					"*"
				]
			},
			"Memory": {
				"measurement": [
					"% Committed Bytes In Use"
				],
				"metrics_collection_interval": 60
			}
		}
	}
}
```
Execute the following command from powershell

```sh
cd C:\Program Files\Amazon\AmazonCloudWatchAgent
.\amazon-cloudwatch-agent-ctl.ps1 -a fetch-config -m ec2 -c file:'C:\Program Files\Amazon\AmazonCloudWatchAgent\config.json' -s
```
Once done, wait for few minutes. The disk and memory usage will be available at AWS account >> CloudWatch >> Metrices >> CWAgent >> AutoScalingGroupName

Create Cloudwatch alarm by selecting any of the metrics.

That's it!
