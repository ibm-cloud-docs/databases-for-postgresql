---

copyright:
  years: 2018, 2023
lastupdated: "2023-05-01"

keywords: pgAdmin, postgresql gui, postgresql, postgres, postgresql cloud database, potgres getting started

subcollection: databases-for-postgresql

content-type: tutorial
services: 
account-plan: paid
completion-time: 30m

---

{{site.data.keyword.attribute-definition-list}}

# Getting Started
{: #getting-started}
{: toc-content-type="tutorial"}
{: toc-services=""}
{: toc-completion-time="30m"}

This tutorial is a short introduction to using an {{site.data.keyword.databases-for-postgresql_full}} deployment. [pgAdmin](https://www.pgadmin.org/){: .external} is an open source administration platform for PostgreSQL, and provides many tools for managing your data and databases. [Download and install](https://www.pgadmin.org/download/){: .external} the version that is appropriate to your environment, and then follow the steps to connect it to your {{site.data.keyword.databases-for-postgresql}} deployment.

## {{site.data.keyword.databases-for-postgresql_full}}
{: #postgresql-product-description}

{{site.data.keyword.databases-for-postgresql_full}} is a serverless, cloud database service that is fully integrated into the {{site.data.keyword.cloud_notm}} environment. You can access and use a cloud database system without purchasing and setting up your own hardware, installing your own database software, or managing the database yourself.

{{site.data.keyword.databases-for-postgresql}} requires no software, infrastructure, network, or OS administration. IBM continuously provides fully automated and automatic updates to the service, such as security and minor version upgrades. A database instance is deployed by default as highly available across multiple data centers in an IBM Cloud Multi-Zone region with synchronous replication. Customers connect to a single database endpoint and IBM automatically manages the failover between Availability Zones. {{site.data.keyword.databases-for-postgresql_full}} can horizontally scale the PostgreSQL instance with read replicas in region or cross-regionally. {{site.data.keyword.databases-for-postgresql}} Read replicas can be easily transformed into fully functioning {{site.data.keyword.databases-for-postgresql}} instances, an especially useful feature for online cross-regional disaster recovery strategies.

Additionally, {{site.data.keyword.databases-for-postgresql}} provides independent scaling of disk, RAM, and vCPU, as well as auto-scaling capabilities and hourly billing. These features help provide greatly increase granularity on right-sizing database use for application workload.

{{site.data.keyword.databases-for-postgresql}} is a multi-tenant offering by design and customers have multiple levers for increased isolation that is detailed in our [Security and Compliance section](/docs/cloud-databases?topic=cloud-databases-manage-security-compliance). For example, configuring a database with vCPUs (referred to as Dedicated Cores) introduces hypervisor-level isolation. Alternatively, if that particular lever of isolation is not necessary, customers can configure databases to pay solely for RAM and disk capacity, without restrictions on movement between these modes. It is an online activity to introduce or remove your usage of Dedicated Cores.

## Before you begin
{: #before-begin}
{: step}

- You need an [{{site.data.keyword.cloud_notm}} account](https://cloud.ibm.com/registration){: .external}.
- You also nneed a {{site.data.keyword.databases-for-postgresql}} deployment. Provision one from the [{{site.data.keyword.cloud_notm}} catalog](https://cloud.ibm.com/catalog/services/databases-for-postgresql). Give your deployment a memorable name that appears in your account's Resource List.
- [Set the Admin Password](/docs/databases-for-postgresql?topic=databases-for-postgresql-admin-password) for your deployment.
- Install [pgAdmin4](https://www.pgadmin.org/download/).
- Review the [`Getting to production`](/docs/cloud-databases?topic=cloud-databases-best-practices) documentation for general guidance on setting up a basic {{site.data.keyword.databases-for-postgresql}} deployment.

## Connect to your database with the CLI
{: #connecting-cli}
{: step}

Find the appropriate commands to connect to your database from the CLI in [Cloud Databases CLI Reference](https://cloud.ibm.com/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference) and [Connecting with psql](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql).

The `ibmcloud cdb deployment-connections` command handles everything that is involved in creating a CLI connection. For example, to connect to a deployment named "example-postgres", use a command like:

```sh
ibmcloud cdb deployment-connections example-postgres --start
```
{: pre}

The command prompts for the admin password and then runs the `psql` CLI to connect to the database. To install the Cloud Databases plug-in, see [Connecting with psql documentation here](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql).

## Connect with pgAdmin
{: #connecting-pgadmin}
{: step}

pgAdmin runs as a server and you connect to it through a browser. When the server is started, it runs on localhost, at default `http://127.0.0.1:53113/browser/`.

When you first open pgAdmin, you get a prompt for setting a primary password. This password is different from your deployment's password as it is used specifically for pgAdmin to store passwords to your PostgreSQL servers or PostgreSQL deployments.

The _Dashboard_ panel has a _Welcome_ screen. From the _Quick Links_, click _Add New Server_.

On your deployment's _Overview_ page, there is an _Endpoints_ panel with all the relevant connection information.

Back in pgAdmin, provide pgAdmin with the information it needs to connect to your deployment. 

First, complete the _Connection_ information, 
- For _Host name/address_, use the _Hostname_ of your deployment.
- For the _Port_, use the _Port_ of your deployment.
- The _Maintenance database_ remains `postgres`.
- For _Username_ and _Password_, use the `admin` credentials that you set after provisioning your deployment. You can choose for pgAdmin to save the password.
- The _Role_ and _Service_ fields can be left empty.

Then, configure the _SSL_ settings.
- Copy the certificate information from the [_Endpoints_ panel](/docs/databases-for-postgresql?topic=databases-for-postgresql-connection-strings) in your deployment's `Dashboard overview` page.
- Save the certificate to a file. (You can use the name that is provided in the download, or your own file name.)
- Set the _SSL mode_ field to _Verify-Full_.
- In the _Root certificate_ field, select the file where you saved your deployment's certificate.

Back on the _General_ tab, give your deployment a name and add any comments that you want to describe or identify your deployment in pgAdmin.

If the _Connect now?_ field is checked, pgAdmin attempts to connect to your deployment when you click the **Save** button.

## Use pgAdmin
{: #using-pgadmin}
{: step}

Once pgAdmin connects, your deployment appears in the _Servers_ list and you get a _Dashboard_ with information and statistics. 

In the list of databases in the _Browser_, there is both the `postgres` database, which you are connected to, and the `ibmclouddb` database, which is the default database for all {{site.data.keyword.databases-for-postgresql}} deployments. Click `ibmclouddb` to connect to it and expand the information about it.

Use pgAdmin to view, administer, and manage your data and databases in your {{site.data.keyword.databases-for-postgresql}} deployment. For more information, see [pgAdmin documentation](https://www.pgadmin.org/docs/pgadmin4/latest/index.html).

Administrative features that require a superuser are not available through pgAdmin because there is no superuser access available to users of a {{site.data.keyword.databases-for-postgresql}} deployment.
{: .tip}

## Next Steps
{: #next-steps}
{: step}

If you are just using PostgreSQL for the first time, see the [official PostgreSQL documentation](https://www.postgresql.org/docs/){: .external}. 

Connect to and manage your databases and data with PostgreSQL's CLI tool [`psql`](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql).

Looking for more tools on managing your deployment? Connect to your deployment with the [IBM Cloud CLI](/docs/cli?topic=cli-install-ibmcloud-cli), the [Cloud Databases CLI plug-in](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference), or the [Cloud Databases API](https://cloud.ibm.com/apidocs/cloud-databases-api).

If you are planning to use {{site.data.keyword.databases-for-postgresql}} for your applications, see [Connecting an external application](/docs/databases-for-postgresql?topic=databases-for-postgresql-external-app) and [Connecting an IBM Cloud application](/docs/databases-for-postgresql?topic=databases-for-postgresql-ibmcloud-app).

To ensure the stability of your applications and your database, see [High-Availability](/docs/databases-for-postgresql?topic=databases-for-postgresql-high-availability) and [Performance](/docs/databases-for-postgresql?topic=databases-for-postgresql-performance).
