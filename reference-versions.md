---
copyright:
  years: 2016,2018
lastupdated: "2018-06-12"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Versions
{: #versions}

Deployable Versions | Preferred Version
----------|-----------
9.6.7, 9.6.9, 10.4 | 10.4
{: caption="Table 1. {{site.data.keyword.databases-for-postgresql}} versions" caption-side="top"}

## Preferred Version

The preferred version is typically the newest version of the PostgreSQL database that is available for {{site.data.keyword.databases-for-postgresql}}. It is the version that is the default in the drop-down version selector on the catalog page. It is also the version that is automatically provisioned by API if no version is specified in the call.

### New Preferred Version Protocol

When a new version is made available, it's release will be announced and it will be available for deployment. Followling release there is an approximately 7-day window where the newest version is available, but it is not the preferred version. This window allows our users to deploy and test the new version, while still having the current version available to them. It will also allow Compose engineers to spot and fix any issues that may arise in the new version. At the end of the 7-day window the new version will be set as the preferred version, or a new date for this change will be announced.

The list of versions available to provision is on the {{site.data.keyword.databases-for-postgresql}} [catalog page](https://console.{{DomainName}}/catalog/services/databases-for-postgresql).

To get a current list of available versions for your {{site.data.keyword.databases-for-postgresql}} service you can use the 
 endpoint.