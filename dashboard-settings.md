---

Copyright:
  years: 2018
lastupdated: "2018-06-25"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Settings

Manage your {{site.data.keyword.databases-for-postgresql_full}} service through these available settings.

## Scale Resources

The _Scale Resources_ panel shows the current size and resource allocation for your service. You can manage resources available to your service by adjusting the individual resources. 

Resources that are unable to be scaled are greyed out. They are present for informational purposes only.
{: .tip} 

### Storage

Disk space allocated based on the size of your data. As you store more data in the service the amount of disk space will increase automatically. Your data is replicated across two data containers in the PostgreSQL cluster, so the amount of space your usage reflects is roughly twice the size of your data. 

### Memory

If you find that your queries and database activity suffers from performance issues due to a lack of memory, you may scale the amount of RAM allocated to your service. Your PostgreSQL service runs with two containers in a cluster, so the amount of memory you add will be added to both containers. 

### Scaling

Scaling operations can cause downtime. When the scaling is complete the Deployment Details pane updates to show the current usage and the new values for the available storage and memory. 

Billing is based on the _total_ amount of resources allocated to the service. 

#### Scaling via the UI

Adjust the slider to raise or lower the resource allocated to your service. Click **Scale** button to trigger the rescaling and return to the dashboard overview.

#### Scaling via the API

Use the `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/groups` endpoint to see and manage your service's resources. Sending a `GET` request will return current resource infromation, including which resources are adjustable. To scale, send a `PATCH` request with the group you are scaling, the resources you are scaling, and the new values for those resources in the body of the request.

#### Scaling via the {{site.data.keyword.cloud_notm}} Databases CLI plug-in

Use the command `ibmcloud dbs groups` to see current resource information for your service, including which resources are adjustable. 

## Setting the admin Password

In order to enable administrative access to your service, you have to set the admin password. You might also find it necessary to change the password of your service. You can do so using Update Password.

A new, randomly generated password appears, or you can type your own password into the field. To regenerate another password, click on the dice to the right of the field. When you click *Update Password* you will be asked to confirm before the change is applied. 

Note: Changing the password changes the credentials that you or any other services that use the admin user to connect. It can cause these services to disconnect and experience downtime.


