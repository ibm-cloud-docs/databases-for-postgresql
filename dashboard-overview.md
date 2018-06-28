---

Copyright:
  years: 2018
lastupdated: "2018-05-11"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Service Overview

The _Overview_ page shows you information about your {{site.data.keyword.databases-for-postgresql_full}} database. The overview includes essential identifying information.

## Deployment Details

The _Deployment Details_ panel shows details of your service.

### Type

The type of database that is offered by the service, and the database version that your service uses.

### ID

An internal identifier for the service.

## Recent Tasks

Making administrative changes to your service (such as scaling, or taking a manual backup) starts a task. While the task is running, the _Recent Tasks_ panel shows the task name and a progress bar. When the task is complete, the _Recent Tasks_ panel lists the most recent previous tasks.

## Instance Administration API

You can manage your {{site.data.keyword.databases-for-postgresql}} service through the {{site.data.keyword.cloud_notm}} Databases API.

### Foundation Endpoint

The foundation endpoint is composed of the region the service resides in and the service instance id. It will be at the start of every endpoint.

### Deployment ID

The deployment ID is necessary for most calls, and identifies the specific deployment instance.

### Reference

For more documentation and reference for using the {{site.data.keyword.cloud_notm}} Databases API, see the {{site.data.keyword.cloud_notm}} Databases API [repository](https://github.ibm.com/compose/apidocs).