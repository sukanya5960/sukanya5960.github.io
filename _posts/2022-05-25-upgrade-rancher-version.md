---
layout: post
title: Upgrade Rancher version
author: Sukanya M
categories: [Kubernetes]
tags: [Kubernetes, devops, EKS]
date: 2022-05-25 14:20:00 +0800
math: true
mermaid: true

---


When you upgrade your Kubernetes cluster version, the kubectl client in Rancher may not be compatible and you need to update Rancher to latest version in order to fix it.

How to upgrade Rancher version If you have launched Rancher server without using an [external DB](https://rancher.com/docs/rancher/v1.1/en/installing-rancher/installing-server/#external-db)

The Rancher server database will be inside the currently running container. We will follow the below steps to upgrade the version.

- First, we will create a data container from running Rancher container.
- Using the data container, start a new Rancher server container by using a --volumes-from

Stop the current Rancher container

```sh
docker stop <container_name_of_original_server>
```

Create a rancher-data container. Note: This step can be skipped if you have already upgraded in the past and already have a rancher-data container.

```sh
docker create --volumes-from <container_name_of_original_server> \
 --name rancher-data rancher/rancher:<tag_of_previous_rancher_server>
```
Pull the most recent image of Rancher

```sh
docker pull rancher/rancher:latest
```

Launch a new Rancher Server container using the database from the rancher-data container. Any changes in Rancher will be saved in the rancher-data container.

```sh
docker run -d --volumes-from rancher-data --privileged \ 
 --restart=always -p 80:80 rancher/rancher:latest
```

Access the Rancher UI and verify the kubectl client version.

If everything is working as expected, remove the old Rancher server container. 

That's it!
