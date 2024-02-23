---

copyright:
  years: 2018, 2024
lastupdated: "2024-02-22"

keywords: deployment, crn, task, gui, api endpoint, postgresql dashboard

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# The Dashboard Overview
{: #dashboard-overview}

The _Overview_ page shows you information about your {{site.data.keyword.databases-for-postgresql_full}} deployment. The overview includes essential identifying information.

## Overview
{: #overview}

### Type
{: #type}

The type of database that is offered by the service, and the database version that your service uses.

### ID
{: #id}

The ID is a [CRN (Cloud Resource Name)](/docs/account?topic=account-crn){: external} that uniquely identifies the database instance. The CRN is used to refer to the database in the API and can be used with the CLI. The _Overview_ pane shows details of your service.

### Recent Tasks
{: #recent-tasks}

Each time that you make administrative changes to your service (such as scaling, or taking a manual backup), a task starts up. The _Recent Tasks_ panel shows the task name and progress bar for any running tasks, and a list of the most recently completed tasks. Depending on how busy your deployment is, successful tasks can be shown for 24 - 48 hours. Unsuccessful tasks can show for 7 - 8 days. Tasks can also be retrieved from the [{{site.data.keyword.databases-for}} API](https://cloud.ibm.com/apidocs/cloud-databases-api#get-currently-running-tasks-on-a-deployment){: external} and [CLI plug-in](https://cloud.ibm.com/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-tasks-list). A historical record of tasks from any time period is available through the [{{site.data.keyword.at_full}} integration](/docs/cloud-databases?topic=cloud-databases-activity-tracker).

### Endpoints
{: #endpoints}

The _Endpoints_ pane within the _Overview_ pane contains connection strings for your instance. Each tab contains connection information that is tailored to the type of connection or the protocol that uses it. Basic information includes _hostname_, _port_, the TLS self-signed certificate, TLS/SSL parameters, and the default database of your instance.

Reference tables for the different connection types are available on the [Getting Credentials and Connection Strings](/docs/databases-for-postgresql?topic=databases-for-postgresql-connection-strings) page.

Connection strings reflect whether your instance uses public endpoints, private endpoints, or both. You can configure which endpoints are available on your deployment. For more information, see the [Service Endpoints Integration](/docs/cloud-databases?topic=cloud-databases-service-endpoints) page.

You can manage your {{site.data.keyword.databases-for-postgresql}} service through the {{site.data.keyword.databases-for}} API. This panel provides the essential information for using the API. For more information about the {{site.data.keyword.databases-for}} API, see the [API reference](https://{DomainName}/apidocs/cloud-databases-api) page.

## Resources
{: #resources-tab}

The resources tab contains information and configuration options on the size and resource usage of your deployment. You can
- [Scale disk, memory, and CPU](/docs/databases-for-postgresql?topic=databases-for-postgresql-resources-scaling)
- [Configure Autoscaling](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling)

## Read Replicas
{: #read-replicas-tab}

The _Read Replicas_ tab contains the UI for details for the read replicas for your deployment. Updates to replication leaders are asynchronously copied to their read-only replicas. For more information, see [Configuring Read-only Replicas](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas).

## Backups
{: #backups-tab}

The _Backups_ tab is the UI for managing your instance's backups. Available backups are listed with their timestamps. Click a backup to grab its ID or to restore it into a new deployment. For more information, see [Managing Backups](/docs/cloud-databases?topic=cloud-databases-dashboard-backups).

## Observability
{: #observability-tab}

The _Observability_ tab provides access to the IBM Cloud Monitoring, logging, and event tracking integrations available for your deployment.
- [{{site.data.keyword.at_full}}](/docs/cloud-databases?topic=cloud-databases-activity-tracker)
- [{{site.data.keyword.la_full}}](/docs/cloud-databases?topic=cloud-databases-logging)
- [{{site.data.keyword.monitoringfull}}](/docs/cloud-databases?topic=cloud-databases-monitoring)

## Settings
{: #settings-tab}

The _Settings_ tab contains the UI for many of the tunable settings for your deployment. You can
- view encryption details. Encryption at rest is enabled for all {{site.data.keyword.databases-for-postgresql}} deployments. If you brought your own encryption key from [Key Protect](/docs/cloud-databases?topic=cloud-databases-key-protect), _Settings_ provides a link to your Key Protect instance, and the _Encryption Key_ field has the name of the key.
- [Change the Admin password](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management&interface=ui#user-management-set-admin-password-ui)
- [Implement or modify an IP allowlist](/docs/cloud-databases?topic=cloud-databases-allowlisting)

## Service Credentials
{: #service-cred-tab}

You can generate a new set of credentials for cases where you want to manually [connect an app](/docs/databases-for-postgresql?topic=databases-for-postgresql-ibmcloud-app) or [external consumer](/docs/databases-for-postgresql?topic=databases-for-postgresql-external-app) to an {{site.data.keyword.cloud_notm}} service. For more information, see [Adding and viewing Credentials](/docs/account?topic=account-service_credentials).

## Connections
{: #connected-resources}

Shows connected resources. Use `Create connection` to bind this service to another resource.

## View docs
{: #view-docs}

The _View docs_ link from the `Actions` drop list opens the main documentation page for {{site.data.keyword.databases-for-postgresql}} in a new tab.
