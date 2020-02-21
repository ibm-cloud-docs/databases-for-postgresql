---

copyright:
  years: 2019, 2020
lastupdated: "2020-02-18"

keywords: postgresql, databases, admin, superuser, roles, service credentials

subcollection: databases-for-postgresql

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}


# Managing Users, Roles, and Privileges 
{: #user-management}

PostgreSQL uses a system of roles to manage database permissions. Roles are used to give a single user or a group of users a set of privileges. You can determine roles, groups, and privileges for all roles across your deployment by using the `psql` command `\du`.

Role name | Attributes | Member of
----------|----------|---------
`admin` | Create role, Create DB | {pg_monitor,pg_signal_backend,ibm-cloud-base-user}
`ibm` | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
`ibm-cloud-base-user` | Create role, Create DB, Cannot login | {}
`ibm-cloud-base-user-ro` | Create role, Create DB, Cannot login | {ibm-cloud-base-user}
`ibm-replication` | Replication | {}
`repl` | Replication | {}
`service_credentials_1` | Create role, Create DB | {ibm-cloud-base-user}
{: caption="Table 1. Users in a PostgreSQL deployment" caption-side="top"}

When you provision a new deployment in {{site.data.keyword.cloud_notm}}, you are automatically given an admin user to access and manage PostgreSQL.

## The `admin` user

When you provision a new deployment in {{site.data.keyword.cloud_notm}}, you are automatically given an admin user to access and manage PostgreSQL. Once you [set the admin password](/docs/services/databases-for-postgresql), you can use it to connect to your deployment.

When admin creates a resource in a database, like a table, admin owns that object. Resources that are created by admin are not accessible by other users, unless you expressly grant permissions to them.

The biggest difference between the admin user and any other users you add to your deployment is the [`pg_monitor`](https://www.postgresql.org/docs/current/default-roles.html) and [`pg_signal_backend`](https://www.postgresql.org/docs/current/default-roles.html) roles. The `pg_monitor` role provides a set of permissions that makes the admin user appropriate for monitoring the database server. The `pg_signal_backend` role provides the admin user the ability to send signals to cancel queries and connections initiated by other users. It does not provide the ability to send signals to processes owned by superusers.

`pg_monitor` is only available in PostgreSQL 10 and above. `pg_signal_backend` is only available in PostgreSQL 9.6 and above.
{: .tip} 

You can also use the admin user to grant these two roles to other users on your deployment.

If you want to expose the ability to cancel queries to other database users, you can grant the `pg_signal_backend` role from the admin user. For example, 
```sql
GRANT pg_signal_backend TO joe;
``` 
to allow the user `joe` to cancel backends. You can also grant `pg_signal_backend` to all the users with the `ibm-cloud-base-user` role with 
```sql
GRANT pg_signal_backend TO "ibm-cloud-base-user";
``` 
Be aware this privilege allows the user or users to terminate any connections to the database, so assign it with care.

Similarly, if you want to setup a specific monitoring user, `mary`, you can use
```
GRANT pg_monitor TO mary;
```
You can also grant `pg_signal_backend` to all the users with the `ibm-cloud-base-user` role with 
```sql
GRANT pg_monitor TO "ibm-cloud-base-user";
```

## _Service Credential_ Users

Users that you [create through the _Service Credentials_ panel](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-connection-strings#creating-users-in-service-credentials) are members of `ibm-cloud-base-user`. They are able to log in, create users, and create databases.

When a user in a group creates a resource in a database, like a table, all users that are in the same group have access to that resource.  Resources created by any of the users in `ibm-cloud-base-user` are accessible to other users in `ibm-cloud-base-user`, including the admin user.

## Users created through the CLI and the API

Users that you create through the Cloud Databases API and the Cloud Databases CLI will also be members of `ibm-cloud-base-user`. They are able to log in, create users, and create databases.

When a user creates a resource in a database, like a table, all users that are in the same group have access to that resource.  Resources created by any of the users in `ibm-cloud-base-user` are accessible to other users in `ibm-cloud-base-user`, including the admin user.

Users that are created directly from the API and CLI do not appear in _Service Credentials_, but you can [add them](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-connection-strings#adding-users-to-_service-credentials_) if you choose.

## The read-only user

The `ibm-cloud-base-user-ro` manages privileges for users that are created to access read-only replicas. More information can be found on the [Configuring Read-only Replicas](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas) page.

## The `repl` user

The `repl` user has Replication privileges and is used if you enable the [`wal2json` plugin](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-wal2json) on your deployment. In the process of enabling `wal2json`, you set the `repl` user's password, which allows the `wal2json` plugin to use it.

## Users created with `psql`

You can bypass creating users through IBM Cloud entirely, and create users directly in PostgreSQL with `psql`. This allows you to make use of PostgreSQL's native [role and user management](https://www.postgresql.org/docs/current/database-roles.html). Users/roles created in `psql` have to have all of their privileges set manually, as well as privileges to the objects that they create. 

Users that are created directly in PostgreSQL do not appear in _Service Credentials_, but you can [add them](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-connection-strings#adding-users-to-_service-credentials_) if you choose. 

Note that these users are not integrated with IAM controls, even if added to _Service Credentials_.
{: .tip}

## The `ibm` and `ibm-replication` Users

If you run the `\du` command with your admin account, you might notice two users that are named `ibm` and `ibm-replication`. These users are internal administrative accounts that manage replication, metrics, and other functions that ensure the stability of your deployment. The `ibm` and the `ibm-replication` accounts are the only superusers on your deployment. A superuser account is not available for you to use.
