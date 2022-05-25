---
layout: post
title: AWS CloudwatchAgent Setup Linux with Autoscaling
author: Sukanya M
categories: [AWS]
tags: [AWS, devops]
date: 2022-05-25 14:20:00 +0800
math: true
mermaid: true

---


In order to monitor memory and disk usage, you need to install Cloudwatch agent in the EC2 instance.

Create IAM role > CloudWatchAgentServerPolicy > and attach it to the instance

Add [CloudWatchAgentAdminPolicy](https://console.aws.amazon.com/iam/home?region=us-east-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FCloudWatchAgentAdminPolicy) and [AmazonEC2RoleforSSM](https://console.aws.amazon.com/iam/home?region=us-east-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2Fservice-role%2FAmazonEC2RoleforSSM)

#### Ubuntu

Execute the folowing commands
```sh
cd /usr/local/src/

wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

dpkg -i amazon-cloudwatch-agent.deb
```

Open the file config.json (if there is no such file, create one)

```sh
cd /opt/aws/amazon-cloudwatch-agent/bin/
vi config.json
```

Copy the following to the config.json file

```sh
{
        "agent": {
                "metrics_collection_interval": 60,
                "run_as_user": "root"
        },
        "metrics": {
                "append_dimensions": {
                        "AutoScalingGroupName": "${aws:AutoScalingGroupName}",
                        "ImageId": "${aws:ImageId}",
                        "InstanceId": "${aws:InstanceId}",
                        "InstanceType": "${aws:InstanceType}"
                },
                "aggregation_dimensions": [["AutoScalingGroupName"]],
                "metrics_collected": {
                        "disk": {
                                "measurement": [
                                        "used_percent",
                                        "inodes_free"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "/"
                                ]
                        },
                        "mem": {
                                "measurement": [
                                        "mem_used_percent"
                                ],
                                "metrics_collection_interval": 60
                        }
                }
        }
}
```
Execute the following command:

```sh
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```

Restart and enable cloudwatch agent to automatically start on boot

```sh
systemctl start amazon-cloudwatch-agent
systemctl enable amazon-cloudwatch-agent
```

Once done, wait for few minutes. The disk and memory usage will be available at AWS account >> CloudWatch >> Metrices >> CWAgent >> AutoScalingGroupName

Create Cloudwatch alarm by selecting any of the metrics.

That's it!
