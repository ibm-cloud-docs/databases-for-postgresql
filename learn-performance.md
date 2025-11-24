---

copyright:
  years: 2020, 2025
lastupdated: "2025-11-24"

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
    {: codeblock}

2. Validate the query blocking
    ```sh
    select pg_blocking_pids(pid), pid AS blocked_pid, state, query, age(now(),query_start) AS duration from pg_stat_activity where array_length(pg_blocking_pids(pid),1) > 0 order by 1,2;
    ```
    {: codeblock}

3. Check the query execution plan.  Run 

    ```sh
    Explain <query>;
    ```
    {: codeblock}

4. Check whether DB stats are missing or not. Use the following query to validate - Action - Run Analyze; or 

    ```sh
    Analyze <tablename>  select relname, n_live_tup, n_dead_tup, last_vacuum,last_autovacuum,last_analyze,last_autoanalyze, analyze_count,autoanalyze_count from pg_stat_all_tables where schemaname='public';
    ```
    {: codeblock}

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
    {: codeblock}

7. Database or objects are created with pg_dump/pg_restore. Action - update stats

8. If Query perform sorting operation, ensure that sufficient work_mem is allocated.-

Additional finding using pg_stat statements

9. Check Query average execution time/ no. of calls

10. CPU % usage by the queries.

11. Memory % usage by the queries.

## Troubleshooting deployment monitoring (disk and memory) and database load monitoring
{: #troubleshooting_monitoring}

{{site.data.keyword.databases-for-postgresql}} deployments offer an integration with the [IBM Cloud Monitoring](docs/cloud-databases?topic=cloud-databases-monitoring) service for monitoring of resource usage on your deployment.

Use {{site.data.keyword.databases-for}} dashboards to set alerts on CPU, memory, and disk IOPS thresholds. Many of the available metrics, like disk usage and IOPS, are useful to help you configure [autoscaling](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling&interface=ui) on your deployment. Autoscaling is not enabled by default and must be configured manually.

Observing trends in your usage and configuring the autoscaling to respond to them can help alleviate performance problems before your databases become unstable due to resource exhaustion, for example changes in disk IO or CPU or memory faults resulting in performance issues.

1. Disk IOPS

The number of input/output operations per second (IOPS) is limited by the type and size of storage volume. Storage volumes for Databases for PostgreSQL deployments are provisioned on [Block Storage Endurance Volumes](/docs/BlockStorage?topic=BlockStorage-orderingBlockStorage&interface=ui) in the 10 IOPS per GB tier.

If your operational load saturates or exceeds the IOPS limit, database requests and operations are delayed until the storage subsystem can catch up. Extended periods of heavy load can cause your deployment to be unable to process queries and become effectively unavailable. You can increase the number IOPS available to your deployment by increasing disk size. For example, for 10 IOPS per GB tier, you can increase IOPS by increasing volume size.

Experience has shown that extended periods of even 40-50% disk utilization have a significant negative impact on database performance.
{: .note}

We recommend at least 100 GB disk (1,000 IOPS) for production environments. IOPS = 10 × allocated GB (for example, 100 GB = 1,000 IOPS)
{: .tip}

2. Memory usage

Memory is the fastest and most efficient way to access and process data. Because of this, a database almost always performs faster using memory instead of reading data from disk. There are other metrics to look at, such as cache hits, blocks read, and blocks hit, so there are other considerations. Just seeing 100% memory usage is not by itself a cause for concern. If memory is exhausted and swap pages are used, performance can degrade significantly.

You can set the amount of memory that is dedicated to the database's shared buffer pool by adjusting [shared_buffers](docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration&interface=cli#mem-settings) in your {{site.data.keyword.databases-for-postgresql}} configuration. The maximum recommended value is 25% of the deployment's total memory. Allocating too much memory to the shared buffer pool can starve the system of memory for other purposes and hinders performance, or possibly even disable the database.

## Database load monitoring
{: #load_monitoring}

There are two options to check database load:

Option 1. Check for long running queries, for example by using {{site.data.keyword.logs_full_notm}} (ICL):
[How do I track query history?](https://cloud.ibm.com/docs/databases-for-postgresql?topic=databases-for-postgresql-pg-queries#troubleshoot-long-term-query-history)

You can use the following sample <DataPrime> query in {{site.data.keyword.logs_full_notm}}:

After the “IBM cloud logs” loads switch to “</>DataPrime” tab (do not change anything in </>lucene search bar). After you switch to </>DataPrime tab, run search as below to get SQL that are running for more than 1000 millis. below is same to paste in search for </>DataPrime tab

```sh
{

source logs|filter message.attr.durationMillis>=1000

}
```
{: codeblock}


* Option 2. Consider enabling the [logminduration_statement](https://www.postgresql.org/docs/14/runtime-config-logging.html)

You can also install the [pg_stat_statements extension](https://cloud.ibm.com/docs/databases-for-postgresql?topic=databases-for-postgresql-extensions)

CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

This allows you to query the stats to find examples of queries that are particularly long-running, so you can isolate issues and consider a different query pattern, new or modified indexes, different table design, or other strategies to improve performance.

### Query 1: Identify time-consuming queries
{: #time_consuming_queries}

You can spot time-consuming queries by running the following command:

```sh
SELECT username, database, queryid, query_preview, calls, total_exec_time, pct_exec_time, ROUND(SUM(pct_exec_time) OVER (ORDER BY total_exec_time DESC)) AS cum_pct_exec_time, avg_exec_time FROM ( SELECT pu.usename AS username, pd.datname AS database, pss.queryid, LEFT(pss.query, 50) AS query_preview, pss.calls, ROUND(pss.total_exec_time::numeric, 3) AS total_exec_time, ROUND((100.0 * pss.total_exec_time / SUM(pss.total_exec_time) OVER ())::numeric, 2) AS pct_exec_time, ROUND(pss.mean_exec_time::numeric, 3) AS avg_exec_time FROM pg_stat_statements pss JOIN pg_user pu ON pss.userid = pu.usesysid JOIN pg_database pd ON pss.dbid = pd.oid ) AS subquery ORDER BY total_exec_time DESC LIMIT 25;
```
{: codeblock}

```sh
username |  database  | queryid |      query_preview       | calls  | total_exec_time | pct_exec_time | cum_pct_exec_time | avg_exec_time 

----------+------------+---------+--------------------------+--------+-----------------+---------------+-------------------+---------------

 ibm      | postgres   |         | <insufficient privilege> |  50685 |       38286.580 |         19.52 |                20 |         0.755

 ibm      | postgres   |         | <insufficient privilege> | 280111 |       28477.951 |         14.52 |                34 |         0.102

 ibm      | postgres   |         | <insufficient privilege> |     18 |       14568.978 |          7.43 |                41 |       809.388

 ibm      | postgres   |         | <insufficient privilege> |     18 |       12103.904 |          6.17 |                48 |       672.439

 ibm      | ibmclouddb |         | <insufficient privilege> |  37552 |        7799.984 |          3.98 |                52 |         0.208

(5 rows)
```
{: codeblock}

This query tracks execution statistics of SQL statements and provides the following information:

* Shows the 25 queries that took the most total time to run

* Displays who ran it, on which database, and a preview of the query

* Shows how often each query was run and how long it took on average

* Calculates what percentage of total database time each query used

* Provides a running total of database time used

### Query 2: Identify frequently run queries
{: #frequently_run_queries}

You can spot frequently run queries by running the following command:

```sh
SELECT
   username,
   database,
   queryid,
   query_preview,
   calls,
   pct_calls,
   ROUND(SUM(pct_calls) OVER (ORDER BY calls DESC)) AS cum_pct_calls,
   total_exec_time,
   avg_exec_time
FROM (
   SELECT
      pu.usename AS username,
      pd.datname AS database,
      pss.queryid,
      LEFT(pss.query, 50) AS query_preview,
      pss.calls,
      ROUND(100.0 * pss.calls / SUM(pss.calls) OVER (), 2) AS pct_calls,
      ROUND(pss.total_exec_time::numeric, 3) AS total_exec_time,
      ROUND(pss.mean_exec_time::numeric, 3) AS avg_exec_time
   FROM
      pg_stat_statements pss
      JOIN pg_user pu ON pss.userid = pu.usesysid
      JOIN pg_database pd ON pss.dbid = pd.oid
) AS subquery
ORDER BY
   calls DESC
LIMIT 25;
```
{: codeblock}


```sh
username |  database  | queryid |      query_preview       | calls | pct_calls | cum_pct_calls | total_exec_time | avg_exec_time 

----------+------------+---------+--------------------------+-------+-----------+---------------+-----------------+---------------

 ibm      | postgres   |         | <insufficient privilege> | 80285 |     23.84 |            24 |       16095.420 |         0.200

 ibm      | postgres   |         | <insufficient privilege> | 36832 |     10.94 |            35 |         548.787 |         0.015

 ibm      | postgres   |         | <insufficient privilege> | 20626 |      6.12 |            41 |         741.755 |         0.036

 ibm      | postgres   |         | <insufficient privilege> | 14702 |      4.36 |            45 |       23982.252 |         1.631

 ibm      | postgres   |         | <insufficient privilege> | 12436 |      3.69 |            49 |        1750.426 |         0.141
```
{: codeblock}

This query provides the following information:

* Shows the 25 queries that were run most often

* Displays who ran it, on which database, and a preview of the query

* Shows how many times each query was run and how long it took in total and on average

* Calculates what percentage of all query calls each query represents
* Provides a running total of query calls



To get current performance across all queries, run the following command:

```sh
select now() as t1,sum(total_exec_time) as et1, sum(calls) as c1 from pg_stat_statements
```
{: codeblock}

```sh
              t1               |        et1         |   c1   

-------------------------------+--------------------+--------

 2025-09-30 08:15:43.898157+00 | 104109.55454499979 | 336889

(1 row)
```
{: codeblock}

then wait 10 seconds and

```sh
select now() as t2,sum(total_exec_time) as et2, sum(calls) as c2 from pg_stat_statements
```
{: codeblock}


```sh
              t2               |        et2         |   c2   

-------------------------------+--------------------+--------

 2025-09-30 08:16:18.943609+00 | 104113.70054399982 | 336896

(1 row)

then -> queries per second = (c2-c1)/(t2-t1) and avg query performance = (et2-et1)/(c2-c1)

To get the top 10 time consuming queries:

```sh
SELECT query, calls, total_exec_time/calls as avg_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```
{: codeblock}

```sh
          query           | calls |      avg_time       

--------------------------+-------+---------------------

 <insufficient privilege> | 14706 |  1.6311896077791408

 <insufficient privilege> | 80305 |  0.2004813740987463

 <insufficient privilege> |     6 |   800.7297508333332

 <insufficient privilege> |     6 |   721.1415835000001

 <insufficient privilege> | 10316 |  0.3907427791779766

 <insufficient privilege> | 10316 | 0.35429842613416024

 <insufficient privilege> | 10316 |  0.3184477990500202

 <insufficient privilege> | 10316 |  0.2285179489143081

 <insufficient privilege> | 12439 |  0.1407493980223488

 <insufficient privilege> | 10316 |  0.1605010507948812

(10 rows)
```
{: codeblock}

pg_stat_activity, pg_stat_bgwriter, and pg_buffercache are important tools in PostgreSQL for monitoring and understanding database performance.
{: .note}


* Check for any recipes running for example backups and batch upload of data:
    Automatic backups are completed daily and kept with a simple retention schedule of 30 days. If a backup got stuck, you can check “Available backups” section and identify the stuck backup in cloud UI page of DB instance.



* Check your IBM Cloud notifications if there was any maintenance for example, database patching

### Possible next actions:
{: #next_actions}

* Optimize slow queries

    * Run EXPLAIN (ANALYZE, BUFFERS) on slow queries. This shows how the database runs the query and where it is slow

```sh
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM student WHERE student_id = 12345;
```
{: codeblock}

* Look for:

    * Queries that use a lot of memory or disk space

    * Add IBM cloud resources as needed:

        * Scale the disk/memory for higher IOPS and memory

* Missing or inefficient indexes are a common cause of slow queries. Use EXPLAIN to identify sequential scans and consider adding indexes

* Run VACUUM to help database health analysis

* Consider using connection pooling to handle more connections. {{site.data.keyword.databases-for-postgresql}} sets the maximum number of connections to your {{site.data.keyword.databases-for-postgresql}} database to **115**.
    15 connections are reserved for the superuser to maintain the state and integrity of your database, and 100 connections are available for you and your applications. After the connection limit is reached, any attempts at starting a new connection result in an error. To prevent overwhelming your deployment with connections, use connection pooling, or scale your deployment and increase its connection limit. For more information, see [Managing PostgreSQL connection pooling](https://cloud.ibm.com/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections#connection-pooling)

* If you believe this is a platform issue such as caused by maintenance, contact IBM Support with the Database CRN
