---

Copyright:
  years: 2018
lastupdated: "2018-07-01"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Connecting an {{site.data.keyword.cloud_notm}} application

Applications running in IBM Cloud have can be bound to your {{site.data.keyword.databases-for-postgresql}} service. 

{{site.data.keyword.cloud_notm}} uses a manifest file - `manifest.yml` to associate an application with a service. Follow these steps to create your manifest file.
- In an editor, open a new file and add the following:
  ```
  ---
  applications:
  - name:    databases-for-postgresql-helloworld-nodejs
    host:    databases-for-postgresql-helloworld-nodejs
    memory:  128M
    services:
      - my-databases-for-postgresql-service
  ```

- Change the host value to something unique. The host you choose will determinate the subdomain of your application's URL: <host>.mybluemix.net.
- Change the name value. The value you choose will be the name of the app as it appears in your {{site.data.keyword.cloud_notm}} dashboard.
- Update the services value to match the name of the service you created in Create a {{site.data.keyword.composeForPostgreSQL}} service instance.

The sample app in the [Getting Started](./getting-started.html) tutorial demonstrates how to use Node.js to bind the service and how to create a database and read from and write to the database.

## Running a cloud application locally

Instead of pushing the app into {{site.data.keyword.cloud_notm}} you can run it locally to test the connection to your {{site.data.keyword.databases-for-postgresql}} service instance. To connect to the service you'll need to create a set of service credentials.

1. From your {{site.data.keyword.cloud_notm}} dashboard, open your {{site.data.keyword.databases-for-postgresql}} service instance.
2. Select _Service Credentials_ from the main menu to open the Service Credentials view.
3. Click **New Credential**.
4. Choose a name for your credentials and click **Add**.
5. Your new credentials are now listed. Click **View credentials** in the corresponding row of the table to view the credentials, and click the **Copy** icon to copy your credentials.
6. In your editor of choice, create a new file with the following, inserting your credentials as shown:

  ```
  {
    "services": {
      "compose-for-postgresql": [
        {
          "credentials": INSERT YOUR CREDENTIALS HERE
        }
      ]
    }
  }
  ```
7. Save the file as `vcap-local.json` in the directory where the sample app is located.

To avoid accidentally exposing your credentials when pushing an application to Github or {{site.data.keyword.cloud_notm}} you should make sure that the file containing your credentials is listed in the relevant ignore file. 
{: .tip}

You can then start your local server.

For information about the credentials you created for the application to connect to your service, see [Using Service Credentials](./connecting-external.html#using-service-credentials).

## Using Service IDs

Since {{site.data.keyword.databases-for-postgresql}} is an IAM service, you can use [Service IDs](https://console.{DomainName}/docs/iam/serviceid.html#serviceids) to manage access to this service. For example, using a service ID that has an API key associated with it will grant that API key to access the {{site.data.keyword.cloud}} Databases API to administrate this service. If you have a Service ID, enter it's information under _Select Service ID_.  


