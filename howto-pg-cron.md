---

copyright:
  years: 2025
lastupdated: "2025-09-25"

keywords: postgresql, databases, pg_cron, schedule, cron, cron jobs, schedule jobs, 

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Scheduling jobs with pg_cron
{: #pg_cron}

`pg_cron` is an in-database job scheduling extension for PostgreSQL (version 10 and above) that lets you automate SQL tasks without leaving the database environment. It follows a familiar cron-like scheduling format, enabling you to set up recurring jobs using standard time expressions. Unlike traditional cron, which runs at the operating system level, pg_cron executes commands directly inside PostgreSQL. It also extends the cron concept by supporting second-level intervals, allowing jobs to run as frequently as every few seconds (from 1 to 59 seconds).

## Setting up `pg_cron`
{: #pg_cron-setting-up}

1. Log in to ibmclouddb database.
   
    ```sh
     \c ibmclouddb
    ```
    {: pre}
   
3. Enable the `pg_cron` extension.
   
    ```sh
     create extension pg_cron;
    ```
    {: pre}
   
5. Verify whether `pg_cron` is installed.
   
    ```sh
     \dx
     ```
   {: pre}
   
    `pg_cron` can be installed on only ibmclouddb database.
   {: note}

7. Run the following command to grant privileges for `pg_cron`.

    ```sh
     select public.grant_pgcron_privileges();
    ```
    {: pre}
   
## Scheduling jobs
{: #pg_cron-schedule-jobs}

`pg_cron` is enabled in ibmclouddb database, so use cron.schedule_in_database() to schedule your jobs.

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

## Examples for using `pg_cron`
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

- Log in into test database to create table:
  
```sh
test=> CREATE TABLE cron_test (id SERIAL PRIMARY KEY, inserted_at TIMESTAMP DEFAULT now() );
CREATE TABLE
```
{: pre}

- Run the following command to schedule a job:
  
```sh
SELECT cron.schedule_in_database('insert-into-cron-test', '* * * * *', $$INSERT INTO cron_test DEFAULT VALUES;$$,'test');

 schedule_in_database
----------------------

                    1
```
{: pre}

- To view scheduled jobs:
  
```sh
ibmclouddb=> select * from cron.job;

 jobid | schedule  |                command                | nodename  | nodeport | database | username | active |        jobname

-------+-----------+---------------------------------------+-----------+----------+----------+----------+--------+-----------------------

     1 | * * * * * | INSERT INTO cron_test DEFAULT VALUES; | localhost |     5432 | test     | admin    | t      | insert-into-cron-test

(1 row)

```
{: pre}

- To view job execution details:

```sh
ibmclouddb=> select * from cron.job_run_details;

 jobid | runid | job_pid | database | username |                command                |  status   | return_message |          start_time           |           end_time

-------+-------+---------+----------+----------+---------------------------------------+-----------+----------------+-------------------------------+-------------------------------

     1 |     1 |    1581 | test     | admin    | INSERT INTO cron_test DEFAULT VALUES; | succeeded | INSERT 0 1     | 2025-08-10 16:21:00.010363+00 | 2025-08-10 16:21:00.026172+00

(1 row)
```
{: pre}

- To unschedule jobs:

```sh
ibmclouddb=> SELECT cron.unschedule(1);
 unschedule
------------
 t
(1 row)
```
{: pre}
