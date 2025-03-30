---
layout: post
title: Blue/Green Deployment in RDS
author: Sukanya M
categories: [AWS]
tags: [AWS, Database, Security]
date: 2024-08-14 20:20:00 +0800
math: true
mermaid: true

---

A blue/green deployment essentially duplicates the live database (Blue) environment into a synchronised replica instance (Green). By using Amazon RDS Blue/Green Deployments, users can make changes to the replica instance without impacting the live environment. This blog outlines the steps for performing a minor version update for a test RDS Instance. 

#### Advantages of blue/green deployment

- Easily create a production-ready staging environment.
- Automatically replicate database changes from the production environment to the staging environment. 
- Test database changes in a safe staging environment without affecting the production environment. 
- Stay current with database patches and system updates. 
- Implement and test newer database features. 
- Switch over your staging environment to be the new production environment without changes to your application. 
- Safely switch over through the use of built-in switchover guardrails. 
- Eliminate data loss during switchover. 
- Switch over quickly, typically under a minute depending on your workload. 

#### Steps

##### Preparing for a blue/green deployment 

Amazon RDS for PostgreSQL primarily uses physical replication for blue/green deployments. For Major version upgrades, it will use logical replication and for minor version upgrade it will use physical replication. Since this is a minor version upgrade, physical replication will be used here. 

- Make sure automated backup is enabled on the DB instance.
- Take a snapshot of the database instance if there is no backup.
- Confirm that the DB instance isn't the source or target of external replication.

##### Creating a blue/green deployment

1. Sign in to the AWS Management Console and open the Amazon RDS console at https://console.aws.amazon.com/rds/ 
1. In the navigation pane, choose Databases, and then choose the DB instance that you want to copy to a green environment.
1. Choose Actions, Create Blue/Green Deployment.
1. Choose the latest minor version for green instance .
1. Verify details and click create. It will take 10-15 minutes to create the blue/green deployment. While creating the blue/green deployment, RDS copies the complete topology and configuration of the primary DB instance (Blue) to create the green environment.
1. RDS will create green instance with latest minor version. User of the blue RDS instances will not be affected by this. Another important thing is green instance will be Read-only.
1. Blue DB instance is copied to the green environment, and RDS configures replication from the DB instance in the blue environment to the DB instance in the green environment. Password to access the green instance will be same as the blue instance. 

##### Testing Application
Any data added the to the blue instance will be replicated to green instance. Test the green instance using it's endpoint. 

During testing, AWS recommend that we keep our databases in the green environment read only. Enable write operations on the green environment with caution because they can result in replication conflicts. They can also result in unintended data in the production databases after switchover. 

If you really need to enable write operation:

- For RDS for PostgreSQL deployments that use logical replication, set the default_transaction_read_only parameter to off at the session level.  

- For those that use physical replication, you can't enable write operations on the green environment.

##### Switch Over
1. Once verified, choose the blue/green deployment that you want to switch over. 
1. For Actions, choose Switch over. The Switch over page appears.
1. On the Switch over page, review the switchover summary. Make sure the   resources in both environments match what you expect. If they don't, choose   Cancel. 
1. For Timeout settings, enter the time limit for switchover. 
1. Choose Switch over.

After a switchover, the DB instances in the previous blue environment are retained. Standard costs apply to these resources. Replication between the blue and green environments stops. 

The endpoint and name of blue instance will be copied to green instance.  

After that, RDS renames the blue instance by appending -oldn to the current resource name, where n is a number.

##### Test the application again 
The green instance became the production instance. Since the RDS instance endpoint is not changed, we don’t need to make any code changes. Verify the application by performing rea/write operations. 

##### Delete the blue/green deployment and blue instance to avoid cost 
You need to remove the old instance to reduce the cost.

##### Roll back 
If the upgrade completes but you encounter application compatibility issues or performance degradation, roll back to the pre-upgrade snapshot. 

- Go to the AWS RDS console. 
- Select Snapshots. 
- Choose the pre-upgrade snapshot. 
- Select Restore snapshot. 
- Provide a new DB instance identifier  
- Configure the restored instance according to your pre-upgrade settings. 
- Restore the parameter groups if they were modified. 
- Point your application to the restored instance. 


#### RDS for PostgreSQL limitations for blue/green deployments with physical replication 
- The following limitations apply to RDS for PostgreSQL blue/green deployments that use physical replication.
  - After the green environment is created, you can't perform a manual major version upgrade.
  - Blue/green deployments that use physical replication don't support schema changes on the green environment, as it is strictly read-only. 
  - The blue DB instance can't be a logical source (publisher) or replica (subscriber).

#### Rollback to Point-in-Time Recovery (PITR)
If you did not take a manual snapshot, and automated backups are enabled, you can use PITR to restore to a point in time just before the upgrade. Point in time recovery creates a new instance, which contains all your data exactly as it existed at the time you selected.

To see the latest restorable time for a DB instance, use the AWS CLI describe-db-instances command and look at the value returned in the LatestRestorableTime field for the DB instance. 

Command: 

```
aws rds describe-db-instances     --db-instance-identifier dbInstanceName| grep LatestRestorableTime 

```
Example:

```
aws rds describe-db-instances     --db-instance-identifier testdb | grep LatestRestorableTime 

Output:           "LatestRestorableTime": "2025-03-19T13:29:33+00:00", 

```

Restoring the database instance using point-in-time recovery backup will create a new instance and you need to point your application to the restored instance once it is verified.