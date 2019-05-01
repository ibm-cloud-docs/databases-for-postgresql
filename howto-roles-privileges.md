---

Copyright:
  years: 2019
lastupdated: "2019-05-01"

subcollection: databases-for-postgresql

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# PostgreSQL Roles and Privileges
{: #roles-privileges}

PostgreSQL uses a system of roles to manage database permissions. Roles are used to give a single user or a group of users a set of privileges. You can determine roles, groups, and privileges for all roles across your deployment by using the `psql` command `\du`.

Role name | Attributes | Member of
----------|----------|---------
`admin` | Create role, Create DB | {pg_monitor,ibm-cloud-base-user}
`ibm` | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
`ibm-cloud-base-user` | Create role, Create DB, Cannot login | {}
`ibm-cloud-base-user-ro` | Create role, Create DB, Cannot login | {ibm-cloud-base-user}
`ibm-replication` | Replication | {}
`service_credentials_1` | Create role, Create DB | {ibm-cloud-base-user}
{: caption="Table 1. Users in a PostgreSQL deployment" caption-side="top"}

When you provision a new deployment in {{site.data.keyword.cloud_notm}}, you are automatically given an admin user to access and manage PostgreSQL. You can also add users in the _Service Credentials_ panel, which allows users for PostgreSQL to be integrated with your {{site.data.keyword.cloud_notm}} account and [IAM](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-iam).

## The `admin` user

When you provision a new deployment in {{site.data.keyword.cloud_notm}}, you are automatically given an admin user to access and manage PostgreSQL. Once you [set the admin password](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-admin-password), you can use it to connect to your deployment.

When admin creates a resource in a database, like a table, admin owns that object. Resources created by admin are not accessible by other users, unless you expressly grant permissions to them.

The biggest difference between the admin user and any other users you add to your deployment is the [`pg_monitor`](https://www.postgresql.org/docs/current/default-roles.html) role. The `pg_monitor` role provides a set of permissions that makes the admin user appropriate for monitoring the database server.

`pg_monitor` is only available in PostgreSQL 10 and above.
{: .tip} 

## The `ibm-cloud-base-user`

You can also add users in the _Service Credentials_ panel, which allows users for PostgreSQL to be integrated with your {{site.data.keyword.cloud_notm}} account and [IAM](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-iam).

Users that you create through the Cloud Databases API and the Cloud Databases CLI will also be members of `ibm-cloud-database-base-user`. They are able to login, create users, and create databases.

When a user creates a resource in a database, like a table, all users that are in the same group have access to that resource.  Resources created by any of the users in `ibm-cloud-base-user` will be accessible to other users in `ibm-cloud-base-user`, including the admin user.

## The read-only user

The `ibm-cloud-base-user-ro` manages privileges for users that are created to access read-only replicas. More information can be found on the [Configuring Read-only Replicas](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas) page.

## `psql` created users

You can create users directly in PostgreSQL with `psql` and make use of PostgreSQL's native [role and user management](https://www.postgresql.org/docs/current/database-roles.html). Users and roles created in `psql` have to have all of their privileges set manually, as well as privileges to the objects that they create.

## The `ibm` and `ibm-replication` Users

If you run the `\du` command with your admin account, you might notice two users named `ibm` and `ibm-replication`. These users are internal administrative accounts that manage replication, metrics, and other functions that ensure the stability of your deployment. The `ibm` and the `ibm-replication` accounts are the only superusers on your deployment. A superuser account is not available for you to use.