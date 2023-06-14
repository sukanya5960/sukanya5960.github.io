---
layout: post
title: Bash Script to Add ISP IP to Security Group
author: Sukanya M
categories: [Programming]
tags: [Bash, AWS, programming]
date: 2023-06-14 20:20:00 +0800
math: true
mermaid: true

---

Suppose, you have an EC2 instance and you need to access it when you are travelling or changing your network provider. Accessing IP restricted AWS Services is a headache as you need to add your ISP IP address to security group everytime your ISP IP address changes (unless you are using a VPN!).

Here is a simple bash script that adds your current ISP IP address to security group for SSH access:

##### Prerequisites
- [Setup AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) - it should have EC2 access
- [Install jq](https://howtoinstall.co/en/jq)

We are going to update an exisiting rule from the security group and the ID of security group and rule are passed as variables.  

![image2](https://raw.githubusercontent.com/sukanya5960/sukanya5960.github.io/master/assets/media/sg.png)



```
#!/bin/bash

region="us-west-2"
sg_id="sg-0ea2b5969193e6cbe"
rule_id="sgr-0769caf4f93569b25"
protocol="TCP"
port=22

# Get the ISP IP address
isp_ip=$(wget -qO- ifconfig.me | awk '{print $1}'| sed 's/$/\/32/')

# Print the ISP IP 
echo "Your ISP IP is $isp_ip"
aws ec2 --region $region modify-security-group-rules \
	    --group-id $sg_id \
	        --security-group-rules SecurityGroupRuleId=$rule_id,SecurityGroupRule='{IpProtocol="TCP",FromPort="'"$port"'",ToPort="'"$port"'",CidrIpv4="'"$isp_ip"'",Description="ISP IP"}'

```

You can copy the script and give execute permission to this. Once you execute the script, it will add your ISP IP address to the SSH inbound rule of the security group.