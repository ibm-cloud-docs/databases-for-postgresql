---
Copyright:
  years: 2018
lastupdated: "2018-07-10"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Managing Access to {{site.data.keyword.databases-for-postgresql}}

{{site.data.keyword.databases-for-postgresql_full}} is an IAM-intergrated service, and provisioning and management of the service is controlled by IAM policies on your account. However, access to the underlying PostgreSQL databases can be controlled through {{site.data.keyword.cloud_notm}}, `psql` and PostgreSQL, or a combination of both. In {{site.data.keyword.cloud_notm}}, you can mangage users through the {{site.data.keyword.cloud_notm}} dashboard, the {{site.data.keyword.cloud_notm}} Databases API, and the {{site.data.keyword.cloud_notm}} CLI databases plug-in. You may also integrate access to your database with IAM-managed Service IDs. For those that prefer a database-native approach, you can directly access the PostgreSQL user management through `psql`.

## Generating a New Credential via _Service Credentials_
To connect an {{site.data.keyword.cloud_notm}} application to {{site.data.keyword.databases-for-postgresql}}, you will need to make at least one service credential. Click on **New Credential** and give it a descriptive name. 

Click **Add** to provision the new Service Credential.

### Using Service IDs

Since {{site.data.keyword.databases-for-postgresql}} is an IAM service, you can use [Service IDs](https://console.{DomainName}/docs/iam/serviceid.html#serviceids) to manage access to this service. For example, using a service ID that has an API key associated with it will grant that API key to access the {{site.data.keyword.cloud}} Databases API to administrate this service. If you have a Service ID, enter it's information under _Select Service ID_. 

### Generating _Service Credentials_ for existing users.

If you have an exisiting PostgreSQL user, created through the API, the IBM Cloud CLI, or through `psql`, and you would like to generate service credentials for them, enter the user name and password in the JSON field below _Add Inline Configuration Parameters_, or specify a file where the JSON information is being stored. For example, `{"existing_credentials":{"username":"Robert","password":"supersecure"}}`. This does not check for or create an associated PostgreSQL user.

## Generating Credentials via the API

To use the API to provision a new credential, send a `POST` request to the `https://api.{region}.databases.cloud.ibm.com/v4/{platform}/deployments/{id}/users` endpoint. Send in the desired username and password in the body of the request.  
For example, the `curl` to create user "mary" with password "mostsecure":
```
curl -X POST "https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users" \
-H "Authorization: Bearer $APITOKEN" \
-H "Content-Type: application/json; charset=utf-8" \
-d \
  '{
  "user": 
    {
      "username":"mary",
      "password":"mostsecure"
    }
  }'
```
Once the credentials have been created, you can view their associated connection information by sending a `GET` request to the`https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/{userid}/connections` endpoint. 

More information can be found in the [API Reference](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html#operation/createDatabaseUser)

## Managing database-level access, `psql`

All users provisioned through the IBM Cloud console, the IBM Cloud databases API, and the IBM Cloud CLI databases plug-in, have the roles `Create role` and  `Create DB`. By default, all provisioned roles have sole dominion over their own resources and database assets created by one role can't be accessed by another.  You can modify the ownership of objects and role membership using `psql`. To view the current state of roles use the `\du` command from `psql`.

You may also create database-level users and roles in `psql` to take full advantage of PostgreSQL's [user management](https://www.postgresql.org/docs/current/static/user-manag.html). However, users created through `psql` will not be managed or visible to the IAM, the IBM Cloud API, or the Service Credentials panel.

### The admin user

By default the service is provisioned with an admin user. At provision time, the password is set but not stored by the provisioning process. The first thing you should do upon provisioning a the service is [set the admin password](./connecting-external.html#setting-the-admin-password).

The admin user comes with the PostgreSQL [default role `pg_monitor`](https://www.postgresql.org/docs/10/static/default-roles.html), allowing access to PostgreSQL monitoring views and functions. By default, the admin user does not have permissions on objects that are created by other users outside of these functions.

For security reasons, and as part of the infrastructure provided by {{site.data.keyword.databases-for-postgresql}} as a managed service, there is no superuser role available to the end-user.
{.tip}