---

copyright:
  years: 2019, 2023
lastupdated: "2023-07-07"

keywords: troubleshooting for PostgreSQL

subcollection: databases-for-postgresql

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

# How do I perform longterm queries?
{: #troubleshoot-long-term-queries}
{: troubleshoot}
{: support}

I want to track long-term queries.
{: shortdesc}

Your {{site.data.keyword.databases-for-postgresql}} deployment is experiencing slow queries. How can you track the history long-term queries to find the root cause?
{: tsSymptoms}

You are experiencing slow queries.
{: tsCauses}

Use the LogDNA timeline feature that that allows you to see how often a search term appears in the log over time. The LogDNA platform provides a timeline feature that allows you to track and visualize the frequency of a specific search term appearing in the log data over time. A long query by itself isn't a problem. A series of them can be, and identifying the actual duration can also help. 

Install and use [`pg_stat_statements`](https://www.postgresql.org/docs/14/pgstatstatements.html){: external} to identify bottlenecks and areas for optimization. `pg_stat_statements` is a PostgreSQL extension that tracks the planning and execution statistics of all SQL statements executed by a server. 

Install and use [`logminduration_statement`](https://www.postgresql.org/docs/14/runtime-config-logging.html){: external}, a configuration parameter in PostgreSQL that determines the minimum length of time after which queries should be logged. It logs the duration of each completed statement if the statement ran for at least the specified number of milliseconds. A value of 0 logs everything and a value of -1 disables logging. This setting can help you track down unoptimized queries in your applications.

For more information, see [Changing your {{site.data.keyword.databases-for-postgresql}} Configuration](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration)
{: tsResolve}


