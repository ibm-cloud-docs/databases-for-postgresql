---

copyright:
  years: 2019, 2023
lastupdated: "2023-11-29"

keywords: databases-for-postgresql release notes

subcollection: databases-for-postgresql

content-type: release-note

---

{{site.data.keyword.attribute-definition-list}}

# Release notes for {{site.data.keyword.databases-for-postgresql_full}}
{: #postgresql-relnotes}

Use these release notes to learn about the latest updates to {{site.data.keyword.databases-for-postgresql_full}} that are grouped by _date_ or _build number_.
{: shortdesc}

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

## 25 Aug 2023
{: #databases-for-postgresql-25aug2023}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} version 11 End of life on February 29, 2024
:  Action required before February 29, 2024, for your PostgreSQL 11 deployments. After February 29, 2024, all {{site.data.keyword.databases-for-postgresql_full}} instances on version 11 that are still active will be disabled. Users are expected to be on the latest versions of PostgreSQL or get the preferred version of {{site.data.keyword.databases-for-postgresql_full}}, with all the great features added to version 15. Upgrade your database instances by following the procedure at [Upgrading to a new Major Version
](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading){: external}.
For more information, see the original [announcement](https://www.ibm.com/blog/announcement/ibm-cloud-databases-for-postgresql-11-end-of-life-on-february-29-2024/){: external}.
If you have any questions or concerns, contact [{{site.data.keyword.databases-for}} support](https://cloud.ibm.com/unifiedsupport/supportcenter){: external}.

## 07 July 2023
{: #databases-for-postgresql-13july2023}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} v15 Preferred
:  For more information, see [Upgrading to a new Major Version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading).

## 25 May 2023
{: #databases-for-postgresql-22may2023}
{: release-note}

Setting up disk alerts for disk utilization tutorial
:  In this tutorial, you use the {{site.data.keyword.cloud_notm}} API and the [{{site.data.keyword.cloud_notm}} CLI](https://cloud.ibm.com/docs/cli?topic=cli-getting-started){: external} to set up an alert that emails you whenever the disk utilization of your database exceeds 90%. This specific example creates an alert on a {{site.data.keyword.databases-for-elasticsearch}} deployment, but it is applicable to all the databases in the IBM {{site.data.keyword.databases-for}} catalog. For more information, see [Setting up disk alerts for disk utilization](/docs/databases-for-postgresql?topic=databases-for-postgresql-disk-util-alert-tutorial).

## 28 October 2022
{: #databases-for-postgresql-28oct2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} v10: End-of-Life Extension
:  To ensure additional time for you to upgrade, {{site.data.keyword.databases-for}} is extending the EOL support for PostgreSQL v10 until February 22, 2023.
Despite the extension, we strongly recommend our customers take prompt action and upgrade instances immediately. If no action is taken, databases will be disabled on February 22, 2023.

## 19 October 2022
{: #databases-for-postgresql-19oct2022}
{: release-note}

Deploying and Connecting a Cloud Databases Instance Tutorial
:  This tutorial guides you through the process of deploying a {{site.data.keyword.databases-for}} instance and connecting it to a web front end by creating a webpage that allows visitors to input a word and its definition. These values are then stored in a database running on {{site.data.keyword.databases-for}}. You install the database infrastructure by using Terraform and your web application uses the popular Express framework. The application can then be run locally, or by using Docker. For more information, see [Deploying and Connecting a {{site.data.keyword.databases-for}} Instance](/docs/databases-for-postgresql?topic=cloud-databases-create-instance-tutorial).

## 11 October 2022
{: #databases-for-postgresql-11oct2022}
{: release-note}

Protecting {{site.data.keyword.databases-for-postgresql_full}} resources with context-based restrictions
:  Context-based restrictions (CBR) give account owners and administrators the ability to define and enforce access restrictions for {{site.data.keyword.cloud}} resources based on the context of access requests. Access to {{site.data.keyword.databases-for}} resources can be controlled with CBR and identity and access management (IAM) policies. For more information, see [Protecting Cloud Databases resources with context-based restrictions](/docs/databases-for-postgresql?topic=cloud-databases-cbr&interface=ui).

## 27 June 2022
{: #databases-for-postgresql-27june2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} 10: End-of-Life in November 2022
:  On 3 November 2022, all {{site.data.keyword.databases-for-postgresql_full}} instances on version 10 that are still active will be disabled. For more information, see [{{site.data.keyword.databases-for-postgresql_full}} 10: End-of-Life in November 2022](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-databases-for-postgresql-10-end-of-life-in-november-2022).

## 31 May 2022
{: #databases-for-postgresql-31may2022}
{: release-note}

Provision a {{site.data.keyword.databases-for-postgresql_full}} instance with Terraform tutorial published
:  In this tutorial, you learn how to use Terraform to provision a {{site.data.keyword.databases-for-postgresql_full}} instance. For more information, see [Provision a {{site.data.keyword.databases-for-postgresql_full}} instance with Terraform](/docs/databases-for-postgresql?topic=cloud-databases-tutorial-provision-postgres-tf).

## 29 March 2022
{: #databases-for-postgresql-29mar2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} supports synchronous replication
:  {{site.data.keyword.databases-for-postgresql_full}} now supports synchronous replication. See documentation [here](/docs/databases-for-postgresql?topic=databases-for-postgresql-high-availability#sync-repl).

## 28 March 2022
{: #databases-for-postgresql-28mar2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} Requirements for upgrading to PostgreSQL (v 13, 14) from PostgreSQL (v10, 11, 12)
:  See information regarding updating to versions 13 and 14 [here](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading#upgrading-reqs).

## 25 March 2022
{: #databases-for-postgresql-25mar2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} Security Goals Updated
:  Available security and compliance goals updated. See updated goals [here](/docs/databases-for-redis?topic=databases-for-redis-manage-security-compliance).

## 30 June 2021
{: #databases-for-postgresql-30jun2021}
{: release-note}

General Availability of {{site.data.keyword.databases-for-postgresql_full}} support for IBM Cloud Databases enabled by IBM Cloud Satellite.
:  A distributed cloud provides consistent security and services across environments, centralized workload visibility, reduced latency, easier compliance, and higher application development velocity. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/deploy-managed-cloud-native-databases-anywhere-with-ibm-cloud-satellite).

## 5 May 2021
{: #databases-for-postgresql-05may2021}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} 9.6 End of Life
:  On 11 November 2021, all IBM Cloud {{site.data.keyword.databases-for-postgresql}} instances on version 9.6 that are still active will be disabled. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-databases-postgresql-9-6-end-of-life).

## 15 February 2021
{: #databases-for-postgresql-15feb2021}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} Horizontal Scaling
:  Customers can scale {{site.data.keyword.databases-for-postgresql_full}} by adding members to their database instance. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/whats-new-in-ibm-cloud-databases).

## 9 Sept 2020
{: #databases-for-postgresql-09sep2020}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} 9.5 End of Life
:  On November 11, 2021, all IBM Cloud {{site.data.keyword.databases-for-postgresql}} instances on version 9.5 that are still active will be disabled. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/postgresql-9-5-end-of-life).

## 13 April 2020
{: #databases-for-postgresql-13apr2020}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} autoscaling
:  Starting after 26 April 2020, all deployments of {{site.data.keyword.databases-for-postgresql_full}} will have a minor change in the format of PostgreSQL logs emitted into Log Analysis with LogDNA.  See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-databases-portfolio-introduces-autoscaling).

## 27 March 2020
{: #databases-for-postgresql-27mar2020}
{: release-note}

Changes to {{site.data.keyword.databases-for-postgresql_full}} Logging
:  We are excited to announce that autoscaling of your deployments based on disk capacity and disk I/O utilization is now available for {{site.data.keyword.databases-for-postgresql_full}}via the UI, API, and CLI. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/changes-to-databases-for-postgresql-logging).

## 2 October 2019
{: #databases-for-postgresql-02oct2019}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} Announces Point-In-Time Recovery
:  {{site.data.keyword.databases-for-postgresql_full}} now allows you to restore a database into a new instance from a specific timestamp. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-databases-for-postgresql-announces-point-in-time-recovery).

## 6 August 2019
{: #databases-for-postgresql-06aug2019}
{: release-note}

New Regions Available for IBM Cloud Database Services
:  {{site.data.keyword.databases-for-postgresql_full}} is now available to be deployed in Seoul; South Korea; and Chennai, India. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/new-regions-available-for-ibm-cloud-database-services).

## 2 October 2018
{: #databases-for-postgresql-02oct2018}
{: release-note}

General Availability of {{site.data.keyword.databases-for-postgresql_full}}
:  {{site.data.keyword.databases-for-postgresql_full}} added to the [IBM Cloud Databases](https://www.ibm.com/cloud/databases) family. See blog post announcement [here](https://www.ibm.com/cloud/blog/ibm-cloud-databases-for-postgresql-and-databases-for-redis-are-now-generally-available).
