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

Your {{site.data.keyword.databases-for-postgresql_full}} backups are accessible from the _Backups_ tab of your service dashboard. Daily, weekly, monthly, and on-demand backups are available. Each backup is labled with its type, and when it was taken. Clicking on it will reveal the backup's full id and a command that you can use to restore a backup with the {{site.data.keyword.cloud_notm}} CLI.

## Restoring a Backup

Restoring a backup will provision a new {{site.data.keyword.databases-for-postgresql}} deployment and restore the data from the backup into the new service. To do so you will need to use the {{site.data.keyword.cloud_notm}} CLI. Use the command `ibmcloud resource service-instance-create SERVICE_INSTANCE_NAME databases-for-postgresql standard us-south -p '{"backup_id":"{backup_id"}`
`SERVICE_INSTANCE_NAME` is whatever you would like to name your new service.

A pre-formatted command for a specific backup is available in detailed view of the backup on the _Backups_ tab of the service dashboard.

## Managing Backups via the API

Use the `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/backups` endpoint to view and manage the available backups for your deployment. Send a `GET` request to return a list of all the backups, including each backup's id. Send a `POST` request to initiate an on-demand backup of your deployment.

For example, the `curl` to list backups:
```
curl -X GET -H "Authorization: Bearer $APIKEY" -H "Content-Type: application/json" "https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/backups"
```
And the `curl` to start an on-demand backup:
```
curl -X POST -H "Authorization: Bearer $APIKEY" -H "Content-Type: application/json" "https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/groups"
```

To get information about a specific backup, send a `GET` request to the endpoint `https://api.{region}.databases.cloud.ibm.com/v4/ibm/backups/{backup_id}`. For example:

```
curl -X GET -H "Authorization: Bearer $APIKEY" -H "Content-Type: application/json" "https://api.{region}.databases.cloud.ibm.com/v4/ibm/backups/{backup_id}"
```
More examples can be found in the [API Reference](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html#tag/Backups)

## Managing Backups via the {{site.data.keyword.cloud_notm}} CLI databases plugin

Use the `dbs deployment-backups-list` command to view the list of all available backups for your deployment. To get the details about a specific backup use `dbs backup-show` command.

For example, to view the backups for a deployment named "example-deployment" use:  
`ibmcloud dbs deployment-backups-list example-deployment`

To see the details of one of the backups from the list use:  
`ibmcloud dbs backup-show crn:v1:staging:public:databases-for-postgresql:us-south:a/6284014dd5b487c87a716f48aeeaf99f:3b4537bf-a585-4594-8262-2b1e24e2701e:backup:a3364821-d061-413f-a0df-6ba0e2951566`

