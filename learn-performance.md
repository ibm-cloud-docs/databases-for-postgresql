---

copyright:
  years: 2020, 2025
lastupdated: "2025-11-20"

keywords: postgresql, databases, monitoring, scaling, autoscaling, resources, postgresql connection limits

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Performance
{: #performance}

{{site.data.keyword.databases-for-postgresql_full}} deployments can be both manually [scaled to your usage](/docs/databases-for-postgresql?topic=databases-for-postgresql-resources-scaling), or configured to [autoscale](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling) under certain resource conditions. There are a few factors to consider when you are tuning the performance of your deployment.

## Monitoring your deployment
{: #monitoring-deployment}

{{site.data.keyword.databases-for-postgresql}} deployments offer an integration with the [{{site.data.keyword.monitoringfull}} service](/docs/cloud-databases?topic=cloud-databases-monitoring) for basic monitoring of resource usage on your deployment. Many of the available metrics, like disk usage and IOPS, are presented to help you configure [autoscaling](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling) on your deployment. Observing trends in your usage and configuring the autoscaling to respond to them can help alleviate performance problems before your databases become unstable due to resource exhaustion.

## Disk IOPS
{: #disk-iops}

The number of input/output operations per second (IOPS) is limited by the type of storage volume. Storage volumes for {{site.data.keyword.databases-for-postgresql}} deployments are provisioned on [Block Storage Endurance Volumes in the 10 IOPS per GB tier](/docs/BlockStorage?topic=BlockStorage-orderingBlockStorage&interface=ui). If your operational load saturates or exceeds the IOPS limit, database requests and operations are delayed until the disk can catch up. Extended periods of heavy-load can cause your deployment to be unable to process queries and become effectively unavailable. If you experience delayed responses and failing operations, you might be exceeding the disk's IOPS limit. You can increase the number IOPS available to your deployment by increasing disk space.

We recommend at least 100 GB disk (1,000 IOPS) for production environments.
{: .tip}

## Memory Usage
{: #mem-usage}

{{site.data.keyword.databases-for-postgresql}} deployment's memory settings are auto-tuned based on the deployment's total memory. Specifically, [`work_mem`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-WORK-MEM), [`maintenance_work_mem`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAINTENANCE-WORK-MEM), and [`effective_cache_size`](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-EFFECTIVE-CACHE-SIZE){: .external} are set on provision, restore, or scale.

You can set the amount of memory that is dedicated to the database's shared buffer pool by adjusting the [`shared_buffers`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-SHARED-BUFFERS){: .external} in your [PostgreSQL configuration](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration){: .external}. The recommended value is 25% of the deployment's total memory. Allocating too much memory to the shared buffer pool can starve the system of memory for other purposes and hinders performance, or possibly even disable the database.

Allocating larger amounts of memory (outside of the shared buffer pool) to your deployment still benefits performance. For example, PostgreSQL fills memory with cached disk pages for performance. It is not necessary to allocate memory to PostgreSQL directly for PostgreSQL to use it.

## Connection limits
{: #connection-limits-performance}

{{site.data.keyword.databases-for-postgresql}} sets the maximum number of connections to your PostgreSQL database to **115**. 15 connections are reserved for the superuser to maintain the state and integrity of your database, and 100 connections are available for you and your applications. After the connection limit is reached, any attempts at starting a new connection results in an error. To prevent overwhelming your deployment with connections, use connection pooling, or scale your deployment and increase its connection limit. For more information, see the [Managing PostgreSQL connections](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections) page.

{{site.data.keyword.databases-for-postgresql}} sets the maximum number of connections to your PostgreSQL database to **115**. 15 connections are reserved for the superuser to maintain the state and integrity of your database, and 100 connections are available for you and your applications. After the connection limit is reached, any attempts at starting a new connection results in an error. To prevent overwhelming your deployment with connections, use connection pooling, or scale your deployment and increase its connection limit. For more information, see the [Managing PostgreSQL connections](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections) page.


## Troubleshooting PostgreSQL performance
{: #troubleshooting_performance}

### Issue: You are seeing slow database performance

The common signs of this issue are as follows:

* High disk IO (input/output) or memory usage
* Slow queries

The most common cause is examining slow queries. Use the following steps to explore:

1. List the long running queries:

    ```sh
    select age(now(),query_start), pid, client_addr, state, substring(query,0,80) from pg_stat_activity where state != 'idle' order by 1 desc;
    ```
    {: pre}

2. Validate the query blocking
    ```sh
    select pg_blocking_pids(pid), pid AS blocked_pid, state, query, age(now(),query_start) AS duration from pg_stat_activity where array_length(pg_blocking_pids(pid),1) > 0 order by 1,2;
    ```
    {: pre}

3. Check the query  execution plan.  Run 

    ```sh
    Explain <query>;
    ```
    {: pre}

4. Check whether DB stats are missing or not. Use the following query to validate - Action - Run Analyze; or 

    ```sh
    Analyze <tablename>  select relname, n_live_tup, n_dead_tup, last_vacuum,last_autovacuum,last_analyze,last_autoanalyze, analyze_count,autoanalyze_count from pg_stat_all_tables where schemaname='public';
    ```
    {: pre}

5. The index might be missing from the table. PostgreSQL provides index methods such as B-tree, hash, GiST, SP-GiST, GIN, and BRIN. Use one of these methods to create the index.

6. The table is bloated: Run below to validate. Action- Run Vaccum <table name> 

    ```sh
    SELECT
    relname AS table_name,
    pg_size_pretty(pg_relation_size(c.oid)) AS actual_size,
    pg_size_pretty(
    CASE
    WHEN c.relpages = 0 THEN 0
    ELSE (pg_relation_size(c.oid) - (c.relpages *
    (current_setting('block_size')::numeric - 24))) * 100 / pg_relation_size(c.oid)
    END
    ) AS bloat_percentage, pg_size_pretty(
    CASE
    WHEN c.relpages = 0 THEN 0
    ELSE (pg_relation_size(c.oid) - (c.relpages *
    (current_setting('block_size')::numeric - 24)))
    END
    )AS bloat_size
    FROM pg_class c
    LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
    WHERE relkind = 'r' AND n.nspname = 'public' -- Adjust schema if needed
    ORDER BY bloat_size DESC NULLS LAST;
    ```
    {: pre}

7. Database or objects are created with pg_dump/pg_restore. Action - update stats

8. If Query perform sorting operation, ensure that sufficient work_mem is allocated.-

9. Check Query average execution time/ no. of calls

10. CPU % usage by the queries.

11. Memory % usage by the queries.

## Troubleshooting deployment monitoring and database load monitoring
{: #troubleshooting_monitoring}

{{site.data.keyword.databases-for-postgresql}} deployments offer an integration with the [IBM Cloud Monitoring](docs/cloud-databases?topic=cloud-databases-monitoring) service for monitoring of resource usage on your deployment.

Use {{site.data.keyword.databases-for}} dashboards to set alerts on CPU, memory, and disk IOPS thresholds. Many of the available metrics, like disk usage and IOPS, are useful to help you configure [autoscaling](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling&interface=ui) on your deployment. Autoscaling is not enabled by default and must be configured manually.

Observing trends in your usage and configuring the autoscaling to respond to them can help alleviate performance problems before your databases become unstable due to resource exhaustion e.g.  changes in disk IO or CPU or memory faults resulting in performance issues.

1. Disk IOPS

The number of input/output operations per second (IOPS) is limited by the type and size of storage volume. Storage volumes for Databases for PostgreSQL deployments are provisioned on Block Storage Endurance Volumes in the 10 IOPS per GB tier. If your operational load saturates or exceeds the IOPS limit, database requests and operations are delayed until the storage subsystem can catch up. Extended periods of heavy-load can cause your deployment to be unable to process queries and become effectively unavailable.  You can increase the number IOPS available to your deployment by increasing disk size e.g. for 10 IOPS per GB tier, users can increase IOPS by increasing volume size.

Note: experience has shown that extended periods of even 40-50% disk utilization have a significant negative impact on database performance.

Tip: We recommend at least 100 GB disk (1,000 IOPS) for production environments. IOPS = 10 × allocated GB (e.g., 100 GB = 1,000 IOPS)

2. Memory usage

Memory is the fastest and most efficient way to access and process data. Because of this, a database almost always performs faster using memory instead of reading data from disk. There are other metrics to look at, such as cache hits, blocks read, and blocks hit, so there are other considerations. Just seeing 100% memory usage is not by itself a cause for concern. If memory is exhausted and swap pages are used, performance can degrade significantly.

You can set the amount of memory that is dedicated to the database's shared buffer pool by adjusting shared_buffers in your PostgreSQL configuration. The maximum recommended value is 25% of the deployment's total memory. Allocating too much memory to the shared buffer pool can starve the system of memory for other purposes and hinders performance, or possibly even disable the database.
