---
layout: post
title: Manually join a Windows instance to Active Directory 
author: Sukanya M
categories: [AWS]
tags: [AWS, devops, Windows]
date: 2022-05-25 14:20:00 +0800
math: true
mermaid: true

---

##### Prerequisites
- [Active directory service provided by AWS](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/ms_ad_getting_started_create_directory.html) should be created 
- Either the EC2 instance launched in same VPC of Active directory or there is VPC peering
- Active directory user login should be created

##### To join a Windows instance to AWS Managed Microsoft AD directory:

1. Connect to the instance using any Remote Desktop Protocol client.

2. Open the TCP/IPv4 properties dialog box on the instance.

    - Open Network Connections.
    - Open the context menu (right-click) for any enabled network connection and then choose **Properties**.
    - In the connection properties dialog box, open **Internet Protocol Version 4**
    
3. Select **Use the following DNS server addresses**, change the **Preferred DNS server** and **Alternate DNS server addresses** to the IP addresses of the AWS Directory Service-provided DNS servers, and choose **OK**.

    ![dns_server_addresses.png](https://raw.githubusercontent.com/sukanya5960/sukanya5960.github.io/master/assets/media/dns_server_addresses.png)

4. Open the **System Properties** dialog box for the instance, select the **Computer Name** tab, and choose **Change**.

5. In the **Member of** field, select **Domain**, enter the fully qualified name of your AWS Directory Service directory, and choose **OK**. When prompted for the name and password for the domain administrator, enter the user name and password of AD user.

6. After you receive the message welcoming you to the domain, restart the instance to have the changes take effect.

That's it!
