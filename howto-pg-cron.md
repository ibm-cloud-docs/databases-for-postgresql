---

copyright:
  years: 2025
lastupdated: "2025-10-31"

keywords: postgresql, databases, pg_cron, schedule, cron, cron jobs, schedule jobs, 

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Scheduling maintenance jobs with pg_cron
{: #pg_cron}

`pg_cron` is a PostgreSQL extension that provides in-database job scheduling, allowing you to automate SQL tasks without relying on external tools. For more information, see [`pg_cron`](https://github.com/citusdata/pg_cron).

The `pg_cron` extension is supported on PostgreSQL version 13 and above.

## Setting up `pg_cron`
{: #pg_cron-setting-up}

1. Log in to the ibmclouddb database as admin user.
   
    ```sh
     \c ibmclouddb
    ```
    {: pre}
   
1. Enable the `pg_cron` extension.
   
    ```sh
     create extension pg_cron;
    ```
    {: pre}
   
1. Verify whether `pg_cron` is installed.
   
    ```sh
     \dx
     ```
   {: pre}
   
    `pg_cron` can be installed on only ibmclouddb databases and can be used only by admin user due to current security reasons.
   {: note}

1. Run the following command to grant privileges for `pg_cron`.

    ```sh
     select public.grant_pgcron_privileges();
    ```
    {: pre}
   
## Scheduling jobs
{: #pg_cron-schedule-jobs}

Use `cron.schedule_in_database()` to schedule your jobs.

```sh
SELECT cron.schedule_in_database(
    job_name text,
    schedule text,          -- cron expression
    command text,           -- SQL command to run
    database_name text      -- target database
);
```
{: pre}

- To view schedule jobs:
  
```sh
select * from cron.job;
```
{: pre}

- To view the status schedule jobs:

```sh
 select * from cron.job_run_details;
```
{: pre}

- To unschedule jobs:

```sh
ibmclouddb=> SELECT cron.unschedule(jobid);
 unschedule
------------
 t
(1 row)
```
{: pre}

## Example for using `pg_cron`
{: #pg_cron-examples}

- Log into ibmclouddb database:

```sh
\c ibmclouddb
```
{: pre}

- Enable the `pg_cron` extension:
  
```sh
ibmclouddb=> \dx
                 List of installed extensions
  Name   | Version |   Schema   |         Description
---------+---------+------------+------------------------------
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
(1 row)

ibmclouddb=> create extension pg_cron;
CREATE EXTENSION
ibmclouddb=>
ibmclouddb=> \dx
                 List of installed extensions
  Name   | Version |   Schema   |         Description
---------+---------+------------+------------------------------
 pg_cron | 1.6     | pg_catalog | Job scheduler for PostgreSQL
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
(2 rows)
```
{: pre}

- Grant privileges for `pg_cron`:
  
```sh
ibmclouddb=> select public.grant_pgcron_privileges();
    grant_pgcron_privileges
--------------------------------
 Granted permission on pg_cron
(1 row)
```
{: pre}


- Scheduling a VACUUM job to run every Sunday at 4:00 AM. 

```sh
SELECT cron.schedule_in_database(
    job_name text,
    schedule text,          -- cron expression
    command text,           -- SQL command to run
    database_name text      -- target database
);
```
  
```sh
 ibmclouddb=> SELECT cron.schedule_in_database('weekly-vacuum', '0 4 * * 0', 'VACUUM', 'test');
 schedule_in_database
----------------------
                   35
(1 row)

```
{: pre}

- To view scheduled jobs:
  
```sh
ibmclouddb=> select * from cron.job;
 jobid | schedule  | command | nodename  | nodeport | database | username | active |    jobname
-------+-----------+---------+-----------+----------+----------+----------+--------+---------------
    35 | 0 4 * * 0 | VACUUM  | localhost |     5432 | test     | admin    | t      | weekly-vacuum
(1 row)

```
{: pre}

- To view job execution details:

```sh
ibmclouddb=> select * from cron.job_run_details;
 jobid | runid | job_pid | database | username | command |  status   | return_message |          start_time           |           end_time
-------+-------+---------+----------+----------+---------+-----------+----------------+-------------------------------+-------------------------------
    35 |    85 |   33810 | test     | admin    | VACUUM  | succeeded | VACUUM         | 2025-09-23 15:48:00.013814+00 | 2025-09-23 15:48:01.29763+00
```
{: pre}

- To unschedule jobs:

```sh
ibmclouddb=> SELECT cron.unschedule(35);
 unschedule
------------
 t
(1 row)
```
{: pre}

- The records in `cron.job_run_details` are not cleaned automatically, but every user that can schedule cron jobs also has permission to delete their own cron.job_run_details records.
  
```sh
ibmclouddb=> SELECT  cron.schedule('delete-job-run-details', '0 12 * * *', $$DELETE FROM cron.job_run_details WHERE end_time < now() - interval '7 days'$$);
```
{: pre}

- `pg_cron` jobs are terminated whenever the database restarts as a result of activities such as scaling, failover, or switchover. Since there is no retry mechanism, the job will not resume automatically. You must either wait for the next scheduled run or adjust the schedule as needed.

- Up to 5 pg_cron jobs run concurrently; extra jobs wait in a queue.
