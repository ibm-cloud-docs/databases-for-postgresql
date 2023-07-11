---

copyright:
  years: 2019, 2023
lastupdated: "2023-07-10"

keywords: troubleshooting for PostgreSQL, query history, slow queries

subcollection: databases-for-postgresql

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

# How do I track query history?
{: #troubleshoot-long-term-query-history}
{: troubleshoot}
{: support}

I want long-term historical tracking of queries, which helps me diagnose issues that I may be having with slow queries.
{: shortdesc}

Your {{site.data.keyword.databases-for-postgresql}} deployment is experiencing slow queries. How can I track query history to find the root cause?
{: tsSymptoms}

You are experiencing slow queries.
{: tsCauses}

- LogDNA is a log management service that can be integrated with {{site.data.keyword.databases-for}} to collect, analyze, and store logs. Use the LogDNA timeline feature to see how often a search term appears in the log over time. The timeline typically shows log events as bars or markers that are distributed along the time axis. The length or height of the bars or markers can represent the event frequency or other parameters, depending on the timeline visualization. A long query by itself isn't a problem. A series of them can be, and identifying the actual duration can also help.
- Enable [`pg_stat_statements`](https://www.postgresql.org/docs/14/pgstatstatements.html){: external} to identify bottlenecks and areas for optimization. `pg_stat_statements` is a PostgreSQL extension that tracks the planning and execution statistics of all SQL statements that are executed by a server.
- Enable [`logminduration_statement`](https://www.postgresql.org/docs/14/runtime-config-logging.html){: external}, a configuration parameter in PostgreSQL that determines the minimum length of time after which queries should be logged. By setting an appropriate value, you can configure PostgreSQL to log queries that exceed a certain execution time. Adjust the value based on your requirements.
- For more information, see [Changing your {{site.data.keyword.databases-for-postgresql}} Configuration](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration).
{: tsResolve}
