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

# Connecting an {{site.data.keyword.cloud_notm}} application

Applications running in {{site.data.keyword.cloud_notm}} can be bound to your {{site.data.keyword.databases-for-postgresql_full}} service. 

{{site.data.keyword.cloud_notm}} uses a manifest file - `manifest.yml` to associate an application with a service. Follow these steps to create your manifest file.
- In an editor, open a new file and add the following:
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

- Change the route value to something unique. The route you choose determines the subdomain of your application's URL: <host>.mybluemix.net.
- Change the name value. The value you choose is the name of the app as it appears in your {{site.data.keyword.cloud_notm}} dashboard.
- Update the services value to match the name or [Cloud Foundry alias](#create-alias) of your {{site.data.keyword.databases-for-postgresql}} service.

You can verify that the services are connected by navigating to the _Connections_ panel. If the service and the application are connected, the connection should show up in both services.

The sample app in the [Getting Started](./getting-started.html) tutorial provides a sample Cloud Foundry application using Node.js and demonstrates to bind the service.

## Creating a Cloud Foundry alias
{: #create-alias}

If your application is running on Cloud Foundry you need to create an alias for your {{site.data.keyword.databases-for-postgresql}} service so that it is discoverable by the Cloud Foundry application. Log into the {{site.data.keyword.cloud_notm}} CLI and use the command:

`ibmcloud resource service-alias alias-name --instance instance-name`

The alias name can be the same as the database service instance name. So, for a {{site.data.keyword.databases-for-postgresql}} service named "example-psql", use the following command:

`ibmcloud resource service-alias example-psql --instance example-psql`





