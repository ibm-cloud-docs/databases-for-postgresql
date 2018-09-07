---
copyright:
  years: 2017,2018
lastupdated: "2017-07-18"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Connecting an external application
{: #connecting-external-app}

{{site.data.keyword.databases-for-postgresql_full}} provides connection string information for you to connect your applications, `psql`, and other third-party PostgreSQL applications. 

## Generating Connection Strings from _Service Credentials_

1. Navigate to the service dashboard for your service.
2. Click _Service Credentials_ to open the _Service Credentials_ panel.
3. Click **New Credential**.
4. Choose a descriptive name for your new credential. 
5. Click **Add** to provision the new credentials. A username and password, and an associated database user in the PostgreSQL database are auto-generated.

### Using Service IDs

Because {{site.data.keyword.databases-for-postgresql}} is an IAM service, you can use [Service IDs](https://console.{DomainName}/docs/iam/serviceid.html#serviceids) to manage access to this service. For example, by using an IAM-managed Service ID, that user gets a PostgreSQL user and connection string in _Service Credentials_, and has API key access to the {{site.data.keyword.cloud_notm}} Databases API.  If you have a Service ID, enter its information under _Select Service ID_.  

### Generating _Service Credentials_ for existing users.

You can generate service credentials for an existing PostgreSQL user that was created through the API, the IBM Cloud CLI, or by using `psql`.

Enter the user name and password in the JSON field _Add Inline Configuration Parameters_, or specify a file where the JSON information is stored. For example, `{"existing_credentials":{"username":"Robert","password":"supersecure"}}`.

Generating credentials from an existing user does not check for or create the associated PostgreSQL user.
{: tip}

## Generating Connection Strings from the command line

Use the `cdb user-create` command to create a new user and password with access to your PostgreSQL deployment. For example, to create a new user for a deployment named "example-deployment", use the following command.

`ibmcloud cdb user-create example-deployment <newusername> <newpassword>`

The response will contain the task `ID`, `Deployment ID`, `Description`, `Created At`, `Status`, and `Progress Percentage` fields.  You can use the task ID to track the progress of user creation with the `cdb task-show` command.

`ibmcloud cdb task-show <taskID>`

## Using Connection Information

Connection information is displayed in the the _Service Credentials_ panel, available from the cloud databases plugin command `cdb deployment-connections --all`, or returned from the API `/users` endpoint. The information that applications and drivers use is in the "PostgreSQL" field. The table describes the connection information.

Field Name|Subfields|Description
----------|-----------
`authentication`|Username, Password, Method|Basic authentication information, username, password, and authentication type.
`certificate`|Name, Base64|A self-signed certificate that is used to confirm that an application is connecting to the appropriate server. It is base64 encoded. You need to decode the key before using it.
`composed`|None|The URI used for connecting to the service. Includes the schema (`postgres:`), admin user name and password, host name of server, port number to connect to, database name and "?sslmode=verify-full" to enable SSL connections and verify the server.
`database`|None|The default database created at deployment provision.
`hosts`|None|The hostname and port of the deployment.
`path`|None|The path to the default database on the deployment.
`query_options`|None|The optional arguments included in the `composed` connection string.
`scheme`|None|The type of database that is offered by the service; in this case `postgresql`.
`type`|None|The format of the `composed` connection string.
{: caption="Table 1. PostgreSQL connection information" caption-side="top"}

## Using the self-signed certificate

The connection information includes a self-signed certificate that you can use in your applications to verify the server when you connect to it. Different drivers for the various languages use various TLS/SSL methods: check the documentation for the driver you are using for specific information on how it handles TLS/SSL.

Usually, you need to complete the following steps.

1. Download and save a copy of the certificate locally
2. Decode it from base64 into a .pem certificate
3. Provide the certificate's path to the driver, and set the SSL mode to "Verify"

### CLI plugin support for the self-signed certificate

You can display the decoded certificate for your deployment with the CLI plugin with the `ibmcloud cdb deployment-cacert your-service-name` command. Copy and save the command's output to a file and provide the file's path to your driver.

## Generating Connection Strings via API

The _Foundation Endpoint_ that is shown on the _Overview_ panel of your service provides the base URL to access this deployment through the API. Use it with the `/users` endpoint if you need to manage or automate user connection string management programmatically.

For more information, see the [API Reference](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html#operation/createDatabaseUser).


 
