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

Your {{site.data.keyword.databases-for-postgresql_full}} backups are accessible from the _Backups_ tab of your service dashboard. Daily, weekly, monthly, and on-demand backups are available. They are retained according to the following schedule:

Backup type|Retention schedule
----------|-----------
Daily|Daily backups are retained for 7 days
Weekly|Weekly backups are retained for 4 weeks
Monthly|Monthly backups are retained for 3 months
On-demand|One on-demand backup is retained. The retained backup is always the most recent on-demand backup.
{: caption="Table 1. Backup retention schedule" caption-side="top"}

## Managing Backups via the API

Use the `https://api.{region}.databases.cloud.ibm.com/v4/{platform}/deployments/{id}/backups` endpoint to view and manage the available backups for your service. Send a `GET` request to return a list of all the backups. Send a `POST` request to initiate an on-demand backup of your service.

To get information about a specific backup, send a `GET` request to the `https://api.{region}.databases.cloud.ibm.com/v4/{platform}/backups/{backup_id}` endpoint. 

## Managing Backups via the {{site.data.keyword.cloud_notm}} CLI databases plugin



## Restoring a Backup