---

copyright:
  years: 2020
lastupdated: "2020-08-24"

keywords: postgresql, databases, pgaudit, logging, session, object, pg role

subcollection: databases-for-postgresql

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# pgAudit
{: #pgaudit}

The PostgreSQL Audit Extension (pgAudit) provides enablement of session logging for your {{site.data.keyword.databases-for-postgresql_full}} deployments. 

### Session Logging

You can configure Session logging parameters that will log all activity for sets of audit event types. Session logging is enabled for the whole DB cluster and is either `on` or `off` for a given event type.

### Event Types

Event types that we support across all versions 
* FUNCTION 
* ROLE
* DDL
* MISC

The additional MISC_SET type is only supported in PostgreSQL 12.
{: .note}

Details of these event types and what they log are here:
https://github.com/pgaudit/pgaudit/blob/master/README.md#pgauditlog

### Enabling pgAudit

pgAudit is enabled directly in the database. No API or CLI access is provided. 

To enable pgAudit, connect as the admin user and call the `set_pgaudit_session_logging` function with the appropriate events.

For example, to enable DDL and ROLE you would call:
```
SELECT public.set_pgaudit_session_logging('{ddl, role}');
```
{: .pre}

Any subsequent calls replace the existing configuration; they are not additive. For example, a subsequent call to `SELECT public.set_pgaudit_session_logging('{misc}');` would result in logging only misc while disabling ddl and role.
{: .note}

### Disabling pgAudit

To disable audit logging: call the same function with `none` specified. For example:
```
SELECT public.set_pgaudit_session_logging('{none}');
```
{: .pre}

Changing audit levels happens immediately when calling the function; it will not interrupt the database activity.
{: .note}

### Audit Logs

Audit events appear in LogDNA with the following format:
```
LOG: AUDIT: SESSION,1,1,DDL,CREATE TABLE,,,create table f2 (id int);,<not logged>
```
The format is documented here https://github.com/pgaudit/pgaudit/blob/master/README.md#format

If you want to see the current log level, you can run the command:  
```
show pgaudit.log;
```
{: .pre
}