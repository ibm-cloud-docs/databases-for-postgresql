---

copyright:
  years: 2017,2018
lastupdated: "2018-06-26"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Backups
{: #backups}

Your {{site.data.keyword.databases-for-postgresql_full}} backups are accessible from the _Backups_ tab of your service dashboard. Daily, weekly, monthly, and on-demand backups are available. 

## Managing Backups via the API

Use the `https://api.{region}.databases.cloud.ibm.com/v4/{platform}/deployments/{id}/backups` endpoint to view and manage the available backups for your service. Send a `GET` request to return a list of all the backups. Send a `POST` request to initiate an on-demand backup of your service.

To get information about a specific backup, send a `GET` request to the endpoint `https://api.{region}.databases.cloud.ibm.com/v4/{platform}/backups/{backup_id}`.

## Managing Backups via the {{site.data.keyword.cloud_notm}} CLI databases plugin

To get information about a specific backup use the `ibmcloud dbs backups-show` command with the service's CRN:backups:backup_id. For example, 
```
ibmcloud dbs backups-show crn:v1:staging:public:databases-for-postgresql:us-south:a/6284014dd5b487c87a716f48aeeaf99f:a2e241f2-8179-4147-8a3b-f4b6caf4c1bb:backups:
```

## Restoring a Backup