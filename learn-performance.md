---

Copyright:
  years: 2020, 2021 
lastupdated: "2021-08-18"

keywords: postgresql, databases, monitoring, scaling, autoscaling, resources, connection limits

subcollection: databases-for-postgresql

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Performance
{: #performance}

{{site.data.keyword.databases-for-postgresql_full}} deployments can be both manually [scaled to your usage](/docs/databases-for-postgresql?topic=databases-for-postgresql-resources-scaling), or configured to [autoscale](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling) under certain resource conditions. There are a few factors to consider if you are tuning the performance of your deployment.

## Monitoring your deployment

{{site.data.keyword.databases-for-postgresql}} deployments offer an integration with the [{{site.data.keyword.monitoringfull}} service](/docs/databases-for-postgresql?topic=databases-for-postgresql-monitoring) for basic monitoring of resource usage on your deployment. Many of the available metrics, like disk usage and IOPS, are presented to help you configure [autoscaling](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling) on your deployment. Observing trends in your usage and configuring the autoscaling to respond to them can help alleviate performance problems before your databases become unstable due to resource exhaustion.

## Disk IOPS

The number of Input-Output Operations Per Second (IOPS) is limited by the type of storage volume. Storage volumes for {{site.data.keyword.databases-for-postgresql}} deployments are provisioned on [Block Storage Endurance Volumes in the 10 IOPS per GB tier](/docs/BlockStorage?topic=BlockStorage-orderingBlockStorage&interface=ui). If your operational load saturates or exceeds the IOPS limit, database requests and operations are delayed until the disk can catch up. Extended periods of heavy-load can cause your deployment to be unable to process queries and become effectively unavailable. If you experience delayed responses and failing operations, you could be exceeding the disk's IOPS limit. You can increase the number IOPS available to your deployment by increasing disk space.

We recommend at least 100 GB disk (1,000 IOPS) for production environments.
{: .tip}

## Memory Usage

{{site.data.keyword.databases-for-postgresql}} deployment's memory settings are auto-tuned based on the deployment's total memory. Specifically, [`work_mem`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-WORK-MEM), [`maintenance_work_mem`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAINTENANCE-WORK-MEM), and [`effective_cache_size`](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-EFFECTIVE-CACHE-SIZE) are set on provision, restore, or scale. 

You can set the amount of memory dedicated to the database's shared buffer pool by adjusting the [`shared_buffers`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-SHARED-BUFFERS) in your [PostgreSQL configuration](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration). The recommended value is 25% of the deployment's total memory. Allocating too much memory to the shared buffer pool can starve the system of memory for other purposes and will hinder performance, or possibly even crash the database.

Allocating larger amounts of memory (outside of the shared buffer pool) to your deployment still benefits performance. For example, PostgreSQL fills memory with cached disk pages for performance. You don't have to allocate memory to PostgreSQL directly for PostgreSQL to use it.

## Connection Limits 
{: #connection-limits-performance}

{{site.data.keyword.databases-for-postgresql}} sets the maximum number of connections to your PostgreSQL database to **115**. 15 connections are reserved for the superuser to maintain the state and integrity of your database, and 100 connections are available for you and your applications. After the connection limit has been reached, any attempts at starting a new connection results in an error. To prevent overwhelming your deployment with connections, use connection pooling, or scale your deployment and increase its connection limit. For more information, see the [Managing PostgreSQL Connections](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections) page.
