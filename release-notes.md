---

copyright:
  years: 2019, 2026
lastupdated: "2026-09-22"

keywords: databases-for-postgresql release notes

subcollection: databases-for-postgresql

content-type: release-note

---

{{site.data.keyword.attribute-definition-list}}

# Release notes
{: #postgresql-relnotes}

Use these release notes to learn about the latest updates to {{site.data.keyword.databases-for-postgresql_full}} that are grouped by _date_ or _build number_.
{: shortdesc}

## 31 July 2026
{: #databases-for-postgreSQL-31Jul2026}
{: release-note}

Built-in support for PgBouncer `auth_query` authentication
:  {{site.data.keyword.databases-for-postgresql}} deployments now include a `pgbouncer_lookup()` helper function and a supporting `pgbouncer_auth` role. If you run your own PgBouncer connection pooler, you can set PgBouncer's `auth_query` option to validate users directly against your deployment instead of maintaining a local user list. Password changes and credential rotations take effect immediately, with no PgBouncer restart required. For more information, see [Connection pooling with PgBouncer](/docs/databases-for-postgresql?topic=databases-for-postgresql-pgbouncer).

## 15 June 2026
{: #databases-for-postgreSQL-15jun2026}
{: release-note}

In-place major version upgrades for {{site.data.keyword.databases-for-postgresql}}
: This release extends in-place major version upgrade (IPMVU) support to PostgreSQL v14, v15, v16, and v17, with PostgreSQL v18 as the target.

:  This enhancement reduces upgrade complexity, minimizes downtime, and ensures a smoother transition to newer versions. Key benefits include:
    * Connection parameters are retained meaning no reconfiguration needed after the upgrade.
    * Faster upgrade process compared to traditional methods.
    * Flexible scheduling so you can choose the upgrade time that best fits your workload.
    * Seamless migration from PostgreSQL v14, v15, v16, or v17 to v18.

:  For more information, see [Upgrading to a new major version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading).

:  You can still perform major version upgrades using read-replica promote upgrade or backup and restore methods. IPMVU provides an additional option.
{: note}

## 27 March 2026
{: #databases-for-postgreSQL-27mar2026}
{: release-note}

Deprecation of {{site.data.keyword.hscrypto}}
: {{site.data.keyword.cloud}} is transitioning its dedicated key management offering from {{site.data.keyword.hscrypto}} to {{site.data.keyword.keymanagementservicelong}} Dedicated (Single Tenant). As part of this transition, {{site.data.keyword.hscrypto}} will reach **End of Life (EOL) on March 20, 2027**. After this date, the service will no longer be supported, and any remaining instances will be terminated.
To ensure continued service availability and support, you must migrate all existing HPCS root keys to {{site.data.keyword.keymanagementservicelong_notm}} Dedicated (Single Tenant) before the EOL date. For more information on how to migrate your encryption keys, see [Migrating from {{site.data.keyword.hscrypto}} (HPCS) to {{site.data.keyword.keymanagementserviceshort}} Dedicated (KP-ST)](/docs/cloud-databases?topic=cloud-databases-hpcs#migrating_hpcs_to_kp).


## 28 Jan 2026
{: #databases-for-postgreSQL-28Jan2026}
{: release-note}

Accelerated availability for PITR restores
:  {{site.data.keyword.databases-for-postgresql}} now allows customers to access database instances immediately following a primary Point-in-Time Recovery (PITR) restoration. This enhancement eliminates the wait time for secondary node synchronization, allowing operations to resume faster than ever.

How it works:

* Accelerated availability: The primary node is restored and opened for traffic, while the secondary (HA) node synchronizes in the background.
* Configuration: This behavior is controlled using the new `async_restore` parameter.

Key benefits and risk considerations:

* This feature enables immediate database access following primary node restoration, significantly reducing the time required to resume operations. However, High Availability (HA) remains inactive during the initial recovery phase; if a zone failure occurs before the secondary node fully synchronizes, the instance may be exposed to potential data loss. Clients are advised to carefully evaluate these risks before enabling this feature.

For implementation guidance, see the [async_restore documentation](/docs/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups&interface=cli#async_restore-pg).

## 18 Dec 2025
{: #databases-for-postgreSQL-18Dec2025}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} version 18 is now available
:  If you use a previous version of {{site.data.keyword.databases-for-postgresql}}, you can migrate to version 18, which is the next available version. For more information, see [Upgrading to a new Major Version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading){: external}.

## 08 Dec 2025
{: #databases-for-postgreSQL-08Dec2025}
{: release-note}

In-place major version upgrades for {{site.data.keyword.databases-for-postgresql}}
:  This new capability simplifies the upgrade process and enhances operational lifecycle management for your database instances.
  * The feature is now available for PostgreSQL v14, providing a streamlined path to v15.
  * Support for additional PostgreSQL versions will be added in upcoming releases.

  This enhancement reduces upgrade complexity, minimizes downtime, and ensures a smoother transition to newer versions. Key benefits include:
  * Connection parameters retained — no reconfiguration needed after the upgrade.
  * Faster upgrade process compared to traditional methods.
  * Flexible scheduling — choose the upgrade time that best fits your workload.
  * Seamless migration from PostgreSQL v14 to v15.

  For more information, see [Upgrading to a new major version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading).

  You can still perform major version upgrades using read-replica promote upgrade or backup and restore methods. The feature provides an additional option.
  {: note}


## 16 October 2025
{: #databases-for-postgresql-16oct2025}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} version 14 End of life on October 21, 2026
:  Action is required before October 21, 2026, for your PostgreSQL v14 deployments. After October 21, 2026, all {{site.data.keyword.cloud_notm}} {{site.data.keyword.databases-for-postgresql}} instances on version 14 that are still active will be upgraded in place to the next major version, version 15. We recommend completing the upgrades before the end-of-life date. For more information, see [Upgrading to a new major version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading).
By proactively upgrading, you can control the timing and minimize any potential downtime. If you have any questions or concerns, contact [{{site.data.keyword.databases-for}} support](https://cloud.ibm.com/unifiedsupport/supportcenter){: external}.


## 02 October 2025
{: #databases-for-postgresql-02oct2025}
{: release-note}

As part of the latest {{site.data.keyword.databases-for-postgresql}} release, two new extensions have been introduced: PgAnon for native data masking and PgCron for job scheduling
:  PgAnon enables efficient data anonymization, supporting privacy and compliance requirements across various use cases. PgCron adds native job scheduling capabilities to PostgreSQL, allowing users to automate recurring tasks within the database environment. For setup instructions and more information, see the documentation for both [anon](/docs/databases-for-postgresql?topic=databases-for-postgresql-data-masking) and [pg_cron](/docs/databases-for-postgresql?topic=databases-for-postgresql-pg_cron).

:  Additionally, this release also includes an enhancement to **Pgaudit**, enabling session logging for your PostgreSQL deployments. For more information, see the [documentation](/docs/databases-for-postgresql?topic=databases-for-postgresql-pgaudit).


## 29 April 2025
{: #databases-for-postgresql-29Apr2025}
{: release-note}

Seamless Vector Search comes to {{site.data.keyword.databases-for-postgresql}} with pgVector
:  {{site.data.keyword.databases-for-postgresql}} users can now enhance their PostgreSQL instances with the pgVector extension, enabling native support for vector embeddings. This tight integration simplifies the development of AI-powered applications, accelerates the process, and reduces architectural complexity by eliminating the need for separate vector databases. For more information, see [Managing PostgreSQL extensions](/docs/databases-for-postgresql?topic=databases-for-postgresql-extensions).

PgVector is only offered for PostgreSQL version 15 and above.
{: important}

## 10 March 2025
{: #databases-for-postgresql-10mar2025}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} {{site.data.keyword.satelliteshort}} plan is deprecated
:   {{site.data.keyword.databases-for-postgresql}} {{site.data.keyword.satelliteshort}} plan is now deprecated. As of March 10 2025, all documentation relating to {{site.data.keyword.databases-for-postgresql}} {{site.data.keyword.satelliteshort}} plan has been removed, as well as the ability to select {{site.data.keyword.databases-for-postgresql}} {{site.data.keyword.satelliteshort}} plan in the Cloud console.

## 15 November 2024
{: #databases-for-postgresql-15nov2024}
{: release-note}

{{site.data.keyword.databases-for}} logs and events are now available on {{site.data.keyword.logs_routing_full}}
: {{site.data.keyword.databases-for}} has onboarded {{site.data.keyword.logs_routing_full}}, a scalable logging service that persists logs and provides users with capabilities for querying, tailing, and visualizing logs. Customers are expected to use {{site.data.keyword.logs_routing_full}} to review their database logs and events starting **November 15, 2024**. For more information, see [Set up logging and monitoring](/docs/databases-for-postgresql?topic=databases-for-postgresql-getting-started-cdb-logging-monitoring) and [About IBM Cloud Logs](/docs/cloud-logs?topic=cloud-logs-about-cl).

## 16 September 2024
{: #databases-for-postgresql-16sept2024}
{: release-note}

Private endpoints as new default
:  To ensure best possible security for your databases, private endpoints are now the default in the {{site.data.keyword.cloud}} console. CLI and Terraform now require the endpoint type to be provided as part of creating an instance.

## 1 May 2024
{: #databases-for-postgresql-01may2024}
{: release-note}

New hosting models
:  You can choose between two hosting models: Isolated Compute and Shared Compute. Isolated Compute is a secure single-tenant offering for complex, highly performant enterprise workloads. Shared Compute is a flexible multi-tenant offering for dynamic, fine-tuned, and decoupled capacity selections. For more information, see [Hosting models](/docs/cloud-databases?topic=cloud-databases-hosting-models&interface=ui){: external}.

## 27 November 2023
{: #databases-for-postgresql-27nov2023}
{: release-note}

Monitoring Integration documentation updated
:  Monitoring Integration documentation now lists metrics for all {{site.data.keyword.databases-for}} services. For more information, see [Monitoring Integration](/docs/cloud-databases?topic=cloud-databases-monitoring){: external}.

## 17 Nov 2023
{: #databases-for-postgresql-25aug2023}
{: release-note}

Deploy pgadmin using Code Engine and connect to your {{site.data.keyword.databases-for-postgresql}} instance published
:  With this tutorial, deploy pgadmin using Code Engine and connect to your {{site.data.keyword.databases-for-postgresql}} instance. pgadmin is a web interface that allows you to view and modify the data in your PostgreSQL database. Code Engine is a a fully managed, serverless platform that allows you to run workloads without worrying about deploying infrastructure.
For more information, see [Deploy pgadmin using Code Engine and connect to your {{site.data.keyword.databases-for-postgresql}} instance](/docs/databases-for-postgresql?topic=databases-for-postgresql-pgadmin-code-engine-icd-postgresql){: external}.

## 07 July 2023
{: #databases-for-postgresql-13july2023}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} v15 Preferred
:  For more information, see [Upgrading to a new Major Version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading){: external}.

## 25 May 2023
{: #databases-for-postgresql-22may2023}
{: release-note}

Setting up disk alerts for disk utilization tutorial
:  In this tutorial, you use the {{site.data.keyword.cloud_notm}} API and the [{{site.data.keyword.cloud_notm}} CLI](https://cloud.ibm.com/docs/cli?topic=cli-getting-started){: external} to set up an alert that emails you whenever the disk utilization of your database exceeds 90%. This specific example creates an alert on a {{site.data.keyword.databases-for-elasticsearch}} deployment, but it is applicable to all the databases in the IBM {{site.data.keyword.databases-for}} catalog. For more information, see [Setting up disk alerts for disk utilization](/docs/cloud-databases?topic=cloud-databases-disk-util-alert-tutorial).

## 19 October 2022
{: #databases-for-postgresql-19oct2022}
{: release-note}

Deploying and Connecting a Cloud Databases Instance Tutorial
:  This tutorial guides you through the process of deploying a {{site.data.keyword.databases-for}} instance and connecting it to a web front end by creating a webpage that allows visitors to input a word and its definition. These values are then stored in a database running on {{site.data.keyword.databases-for}}. You install the database infrastructure by using Terraform and your web application uses the popular Express framework. The application can then be run locally, or by using Docker. For more information, see [Deploying and Connecting a {{site.data.keyword.databases-for}} Instance](/docs/cloud-databases?topic=cloud-databases-create-instance-tutorial).

## 11 October 2022
{: #databases-for-postgresql-11oct2022}
{: release-note}

Protecting {{site.data.keyword.databases-for-postgresql}} resources with context-based restrictions
:  Context-based restrictions (CBR) give account owners and administrators the ability to define and enforce access restrictions for {{site.data.keyword.cloud}} resources based on the context of access requests. Access to {{site.data.keyword.databases-for}} resources can be controlled with CBR and identity and access management (IAM) policies. For more information, see [Protecting Cloud Databases resources with context-based restrictions](/docs/cloud-databases?topic=cloud-databases-cbr&interface=ui).

## 31 May 2022
{: #databases-for-postgresql-31may2022}
{: release-note}

Provision an {{site.data.keyword.databases-for-postgresql}} instance with Terraform tutorial published
:  In this tutorial, you learn how to use Terraform to provision an {{site.data.keyword.databases-for-postgresql}} instance. For more information, see [Provision an {{site.data.keyword.databases-for-postgresql}} instance with Terraform](/docs/cloud-databases?topic=cloud-databases-tutorial-provision-postgres-tf&interface=ui).

## 29 March 2022
{: #databases-for-postgresql-29mar2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} supports synchronous replication
:  {{site.data.keyword.databases-for-postgresql}} now supports synchronous replication. See documentation [here](/docs/databases-for-postgresql?topic=databases-for-postgresql-postgresql-ha-dr#ha-synchronous-replication).

## 25 March 2022
{: #databases-for-postgresql-25mar2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} Security Goals Updated
:  Available security and compliance goals updated. See updated goals [here](/docs/cloud-databases?topic=cloud-databases-manage-security-compliance).

## 30 June 2021
{: #databases-for-postgresql-30jun2021}
{: release-note}

General Availability of {{site.data.keyword.databases-for-postgresql}} support for {{site.data.keyword.cloud_notm}} Databases enabled by {{site.data.keyword.cloud_notm}} Satellite.
:  A distributed cloud provides consistent security and services across environments, centralized workload visibility, reduced latency, easier compliance, and higher application development velocity.

## 15 February 2021
{: #databases-for-postgresql-15feb2021}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} Horizontal Scaling
:  Customers can scale {{site.data.keyword.databases-for-postgresql}} by adding members to their database instance.

## 13 April 2020
{: #databases-for-postgresql-13apr2020}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} autoscaling
:  Starting after 26 April 2020, all deployments of {{site.data.keyword.databases-for-postgresql}} will have a minor change in the format of PostgreSQL logs emitted into Log Analysis with LogDNA.

## 27 March 2020
{: #databases-for-postgresql-27mar2020}
{: release-note}

Changes to {{site.data.keyword.databases-for-postgresql}} Logging
:  We are excited to announce that autoscaling of your deployments based on disk capacity and disk I/O utilization is now available for {{site.data.keyword.databases-for-postgresql}}via the UI, API, and CLI.

## 2 October 2019
{: #databases-for-postgresql-02oct2019}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} Announces Point-In-Time Recovery
:  {{site.data.keyword.databases-for-postgresql}} now allows you to restore a database into a new instance from a specific timestamp.

## 6 August 2019
{: #databases-for-postgresql-06aug2019}
{: release-note}

New Regions Available for {{site.data.keyword.cloud_notm}} Database Services
:  {{site.data.keyword.databases-for-postgresql}} is now available to be deployed in Seoul; South Korea; and Chennai, India.

## 2 October 2018
{: #databases-for-postgresql-02oct2018}
{: release-note}

General Availability of {{site.data.keyword.databases-for-postgresql}}
:  {{site.data.keyword.databases-for-postgresql}} added to the [{{site.data.keyword.cloud_notm}} Databases](https://www.ibm.com/cloud/databases) family.
