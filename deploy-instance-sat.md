---
copyright:
  years: 2021
lastupdated: "2021-06-21"

keywords: postgresql, databases, instance

subcollection: databases-for-postgresql

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:important: .important}

# Assigning storage to service cluster
{: #instance-sat}

**Create** an instance in the Satellite location and verify that a cluster has been created and the instance is provisioning. 
When the service cluster is created, if the corresponding hosts are available then the hosts are typically **auto-assigned** to the cluster.

## Cluster Configuration

To assign an existing Satellite storage configuration to IBM Cloud™ Databases for PostgreSQL enabled by IBM Cloud Satellite service, the Satellite location administrator must perform the following manual step to fetch the appropriate information from the **application developer**.
{: important}

- This should be done using the Satellite CLI, as shown below:

```
ibmcloud sat storage assignment create \
  --name 'assignment-lsacj8oegr' \
  --config 'b1005956-2d9d-4d46-9fbf-0589c9437716' \
  --cluster 'c2tjvgpw0mdf25k6m040'
```
{: pre}

- Once this step is complete, the next step is to deploy the requested database instance.
