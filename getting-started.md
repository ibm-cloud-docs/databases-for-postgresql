---

copyright:
  years: 2018
lastupdated: "2018-07-31"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}


# Getting started tutorial
This tutorial uses the [clouddatabases-postgresql-helloworld-nodejs](https://github.com/IBM-Cloud/clouddatabases-postgresql-helloworld-nodejs) sample app to demonstrate how to connect a Cloud Foundry application in {{site.data.keyword.cloud_notm}} to an {{site.data.keyword.databases-for-postgresql_full}} service. The application creates, reads from, and writes to a database using data supplied through the app's web interface.
{: shortdesc}

## Before you begin

Make sure you have an [{{site.data.keyword.cloud_notm}} account][ibm_cloud_signup_url]{:new_window}.

You'll also need to install [Node.js](https://nodejs.org/) and [Git](https://git-scm.com/downloads).

## Step 1: Create a {{site.data.keyword.databases-for-postgresql}} service instance
{: #create-service}

You can create a {{site.data.keyword.databases-for-postgresql}} service from the [{{site.data.keyword.databases-for-postgresql}} page](https://console.{DomainName}/catalog/services/databases-for-postgresql/) in the {{site.data.keyword.cloud_notm}} catalog.

Choose a service name, region, organization and space to provision the service in, and for the **Select a database version** field, choose _Latest Preferred Version_. In this example, the service name is "example-psql".

Click **Create** to provision your service. You will be taken back to your {{site.data.keyword.cloud_notm}} _Dashboard_ while the service is provisioning Provisioning can take a while to complete.

You won't be able to connect an application to the service until provisioning has completed.
{: .tip}

## Step 2: Clone the Hello World sample app from Github

Clone the Hello World app to your local environment from your terminal using the following command:

```
git clone https://github.com/IBM-Cloud/clouddatabases-postgresql-helloworld-nodejs.git
```

## Step 3: Install the app dependencies

Use npm to install dependencies.

1. From your terminal, change the directory to where the sample app is located.
  
  ```
  cd clouddatabases-postgresql-helloworld-nodejs
  ```

2. Install the dependencies listed in the `package.json` file.
  
  ```
  npm install
  ```

## Step 4: Download and install the {{site.data.keyword.cloud_notm}} CLI tool

The {{site.data.keyword.cloud_notm}} CLI tool tool is what you'll use to communicate with {{site.data.keyword.cloud_notm}} from your terminal or command line. For details, see [Download and install {{site.data.keyword.cloud_notm}} CLI](https://console.{DomainName}/docs/cli/reference/bluemix_cli/download_cli.html).

## Step 5: Connect to {{site.data.keyword.cloud_notm}}

1. Connect to {{site.data.keyword.cloud_notm}} in the command line tool and follow the prompts to log in.

  ```
  ibmcloud login
  ```

  If you have a federated user ID, use the `ibmcloud login --sso` command to log in with your single sign on ID. See [Logging in with a federated ID](https://console.{DomainName}/docs/cli/login_federated_id.html#federated_id) to learn more.
  {: .tip}

2. Make sure you are targeting the correct {{site.data.keyword.cloud_notm}} org and space.

  ```
  ibmcloud target --cf
  ```

  Choose from the options provided, using the same values you used when you created the service.

## Step 6: Create a Cloud Foundry alias for the database service.
{: #create-alias}

Make the database service discoverable by Cloud Foundry applications by giving it a Cloud Foundry alias. 

`ibmcloud resource service-alias-create alias-name --instance-name instance-name`

The alias name can be the same as the database service instance name. So, for our database created in step 1, we could do:

`ibmcloud resource service-alias-create example-psql --instance-name example-psql`

## Step 7: Update the app's manifest file
{: #update-manifest}

{{site.data.keyword.cloud_notm}} uses a manifest file - `manifest.yml` to associate an application with a service. Follow these steps to create your manifest file.

1. In an editor, open a new file and add the following:

  ```
  ---
  applications:
  - name:    example-helloworld-nodejs
    routes:
    - route: example-helloworld-nodejs.mybluemix.net
    memory:  128M
    services:
      - example-psql
  ```

2. Change the `route` value to something unique. The host you choose will determinate the subdomain of your application's URL:  `<route>.mybluemix.net`.
3. Change the `name` value. The value you choose will be the name of the app as it appears in your {{site.data.keyword.cloud_notm}} dashboard.
4. Update the `services` value to match the alias of the service you created in [Create a Cloud Foundry alias for the database service](#create-alias).

## Step 8: Push the app to {{site.data.keyword.cloud_notm}}.

This step will fail if the service has not finished provisioning from Step 1. You can check to see if it has completed on your {{site.data.keyword.cloud_notm}} _Dashboard_.
{: .tip}

When you push the app it will automatically be bound to the service specified in the manifest file.

```
ibmcloud cf push
```

## Step 9: Check the app is connected to your {{site.data.keyword.databases-for-postgresql}} service

1. Navigate to your {{site.data.keyword.databases-for-postgresql}} service dashboard
2. Select _Connections_ from the dashboard menu. Your application should be listed under _Connected Applications_.

If your application is not listed, repeat Steps 7 and 8, making sure you have entered the correct details in [manifest.yml](#update-manifest).

## Step 10: Use the app

Now, when you visit `<route>.mybluemix.net/` you will be able to view the contents of your {{site.data.keyword.databases-for-postgresql}} collection. As you add words and their definitions they are added to the database and displayed. If you stop and restart the app you'll see any words and definitions you've already added are now listed.

## Running the app locally

Instead of pushing the app into {{site.data.keyword.cloud_notm}} you can run it locally and still connect to your {{site.data.keyword.databases-for-postgresql}} service instance. To connect to the service you need to create a set of service credentials.

1. From your {{site.data.keyword.cloud_notm}} dashboard, open your {{site.data.keyword.databases-for-postgresql}} service instance.
2. Select _Service Credentials_ from the main menu to open the Service Credentials view.
3. Click **New Credential**.
4. Choose a name for your credentials and click **Add**.
5. Your new credentials are now listed. Click **View credentials** in the corresponding row of the table to view the credentials, and click the **Copy** icon to copy your credentials.
6. In your editor of choice, create a new file with the following, inserting your credentials as shown:

  ```
  {
    "services": {
      "databases-for-postgresql": [
        {
          "credentials": INSERT YOUR CREDENTIALS HERE
        }
      ]
    }
  }
  ```
7. Save the file as `vcap-local.json` in the directory where the sample app is located.

To avoid accidentally exposing your credentials when pushing an application to Github or {{site.data.keyword.cloud_notm}}, make sure that the file containing your credentials is listed in the relevant ignore file. 
{: .tip}

You can then start the local server.  
`npm start`
The app is now running at http://localhost:8080. You can add words and definitions to your {{site.data.keyword.databases-for-postgresql}} database. When you stop and restart the app, any words you have already added are displayed when you refresh the page.

For information about the credentials you created for the application to connect to your service, see [Using Service Credentials](./connecting-external.html#using-service-credentials).

## Next steps

To understand more about how the [clouddatabases-postgresql-helloworld-nodejs](https://github.com/IBM-Cloud/clouddatabases-postgresql-helloworld-nodejs) sample app works, you can read the application's readme file, or the code comments in `server.js`, which give some information about the app's functions.

To start exploring your {{site.data.keyword.databases-for-postgresql}} service, see the following topics about the service dashboard:

- [Dashboard Overview](./dashboard-overview.html)
- [Backups](./dashboard-backups.html)
- [Settings](./dashboard-settings.html)
