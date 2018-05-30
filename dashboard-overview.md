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

The _Overview_ page shows you information about your {{site.data.keyword.cloud}} database. The overview includes essential identifying information and current resource usage. You'll also find a section for connection strings that you can use with tools or make use of tools to connect to your database.

## Deployment Details

The _Deployment Details_ panel shows details of your service.

![Deployment Details](./images/postgres-deployment-details.png "A view of the Deployment Details panel")

### Type

The type of database that is offered by the service, and the database version that your service uses.

### ID

An internal identifier for the service.

### Usage

The size of your database and the amount of storage provided by your service plan.

## Current Jobs

Making administrative changes to your service (such as scaling, or taking a manual backup) starts a job. While a job is running, the _Current Jobs_ panel appears on the _Overview_ page, showing the job name and a progress bar. When the job is complete, the _Current Jobs_ panel no longer appears on the _Overview_ page.

## Instance Administration API

You can manage your ICD for PostgreSQL service through the {{site.data.keyword.cloud_notm}} Compose API.

### Foundation Endpoint

The foundation endpoint is composed of the region the service resides in and the service instance id. It will be at the start of every endpoint.

### Deployment ID

The deployment ID is necessary for most calls, and identifies the specific deployment instance.

### Reference

For more documentation and reference for using the {{site.data.keyword.cloud_notm}} Compose API, across all {{site.data.keyword.cloud_notm}} Compose services, read [The {{site.data.keyword.cloud_notm}} Compose API](https://www.compose.com/articles/the-ibm-cloud-compose-api/).