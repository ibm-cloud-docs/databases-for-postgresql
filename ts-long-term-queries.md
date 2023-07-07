---

copyright:
  years: 2019, 2023
lastupdated: "2023-07-07"

keywords: troubleshooting for PostgreSQL

subcollection: databases-for-postgresql

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

# How do I track long-term query history?
{: #troubleshoot-long-term-query-history}
{: troubleshoot}
{: support}

I want to track the history of long-term queries.
{: shortdesc}

Your {{site.data.keyword.databases-for-postgresql}} deployment is experiencing slow queries. How can I track the history long-term queries to find the root cause?
{: tsSymptoms}

You are experiencing slow queries.
{: tsCauses}

- LogDNA is a log management service that can be integrated with IBM Cloud Databases to collect, analyze, and store logs. Use the LogDNA timeline feature that allows you to see how often a search term appears in the log over time. You will see a graphical representation of log events over time. The timeline typically shows log events as bars or markers distributed along the time axis. The length or height of the bars/markers can represent the event frequency or other parameters, depending on the timeline visualization. A long query by itself isn't a problem. A series of them can be, and identifying the actual duration can also help. 
- Enable [`pg_stat_statements`](https://www.postgresql.org/docs/14/pgstatstatements.html){: external} to identify bottlenecks and areas for optimization. `pg_stat_statements` is a PostgreSQL extension that tracks the planning and execution statistics of all SQL statements executed by a server.
- Enable [`logminduration_statement`](https://www.postgresql.org/docs/14/runtime-config-logging.html){: external}, a configuration parameter in PostgreSQL that determines the minimum length of time after which queries should be logged. By setting an appropriate value, you can configure PostgreSQL to log queries that exceed a certain execution time. Adjust the value based on your requirements.
- For more information, see [Changing your {{site.data.keyword.databases-for-postgresql}} Configuration](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration).
{: tsResolve}
