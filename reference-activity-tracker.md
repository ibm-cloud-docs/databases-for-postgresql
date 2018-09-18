---
copyright:
  years: 2017,2018
lastupdated: "2018-09-17"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Activity Tracker Integration

{{site.data.keyword.databases-for-postgresql_full}} is integrated with  [Activity Tracker]
(https://console.{DomainName}/docs/services/cloud-activity-tracker/activity_tracker_ov.html#activity_tracker_ov), so you can view service-level events.

In order to see the events, you need to [provision the Activity Tracker service](https://console.{DomainName}/docs/services/cloud-activity-tracker/how-to/provision.html#provision) from the [{{site.data.keyword.cloud_notm}}  catalog](https://console.{DomainName}/catalog/services/activity-tracker). Activity Tracker has a _Lite_ plan available at no additional cost.

Once you have provisioned Activity Tracker, the {{site.data.keyword.databases-for-postgresql}} events will appear under _Account Logs_ from the _View Logs_ dropdown menu. 

In addition to the Activity Tracker UI, the Kibana dashboard is also available at https://logging.ng.bluemix.net.
{: .tip}

## Event Fields
A description of the common fields for an Activity Tracker event is on the [Activity Tracker event fields](https://console.{DomainName}/docs/services/cloud-activity-tracker/at_event.html#at_event) page.

## List of Events

The table lists the events that get sent to Activity Tracker from {{site.data.keyword.databases-for-postgresql}}.

Action|Message|Description
-------|-------
deployment.backup|Databases for PostgreSQL: backup deployment|A backup was created.
deployment.backup|Databases for PostgreSQL: backup deployment -failure|A backup failed.
deployment-user-password.update|Databases for PostgreSQL: update deployment-user-password|A user's password was updated
deployment-user-password.update|Databases for PostgreSQL: update deployment-user-password -failure|An attempt to update a user's password failed.
deployment-user.create|Databases for PostgreSQL: create deployment-user|A user was created.
deployment-user.create|Databases for PostgreSQL: create deployment-user -failure|An attempt to create a user failed.
deployment-user.delete|Databases for PostgreSQL: delete deployment-user|A user was deleted.
deployment-user.delete|Databases for PostgreSQL: delete deployment-user -failure|An attempt to delete a user failed.
deployment-connections.renew|Databases for PostgreSQL: renew deployment-connections|
deployment-connections.renew|Databases for PostgreSQL: renew deployment-connections -failure|
deployment.create|Databases for PostgreSQL: create deployment|A deployment was created.
deployment.create|Databases for PostgreSQL: create deployment -failure|An attempt to create a deployment failed.
deployment.restore|Databases for PostgreSQL: restore deployment|A restore from backup was created.
deployment.restore|Databases for PostgreSQL: restore deployment -failure|An attempted restore from backup failed.
deployment.scale|Databases for PostgreSQL: scale deployment|A scaling operation was performed.
deployment.scale|Databases for PostgreSQL: scale deployment -failure|An attempted scaling operation failed.
deployment-whitelisted-ips-list.update|Databases for PostgreSQL: update deployment-whitelisted-ips-list|The whitelist was modified.
deployment-whitelisted-ips-list.update|Databases for PostgreSQL: update deployment-whitelisted-ips-list -failure|An attempt to modify the whitelist failed.
{: caption="Table 1. Event Descriptions " caption-side="top"}