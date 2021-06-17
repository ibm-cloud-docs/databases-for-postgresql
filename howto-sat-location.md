---

copyright:
  years: 2015, 2020
lastupdated: "2020-11-12"

keywords: IBM Cloud, databases, Satellite, ICD

subcollection: databases-for-postgresql

---

{:shortdesc: .shortdesc}
{:screen: .screen}  
{:codeblock: .codeblock}  
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:external: target="_blank" .external}

# Getting started with IBM Cloud™ Databases Satellite
{: #getting-started}

<!-- The title of your H1 should be Getting started with _service-name_, where _service-name_ is the non-trademarked short version conref. -->

Set up an IBM Cloud Databases for PostgreSQL enabled by IBM Cloud Satellite location to represent a data center that you fill with your own infrastructure resources, and start running IBM Cloud™ Databases on your own infrastructure.{: shortdesc}

## Before you begin
{: #prereqs}
- You need to have an [{{site.data.keyword.cloud_notm}} account](https://cloud.ibm.com/registration){:new_window}.
- You will also need a {{site.data.keyword.databases-for-postgresql}} deployment. You can provision one from the [{{site.data.keyword.cloud_notm}} catalog](https://cloud.ibm.com/catalog/services/databases-for-postgresql). Give your deployment a memorable name that appears in your account's Resource List.
- [Set the Admin Password](/docs/databases-for-postgresql?topic=databases-for-postgresql-admin-password) for your deployment.

## Setting Up a Satellite Service 

1. Log in to your [{{site.data.keyword.cloud_notm}} account](https://cloud.ibm.com/registration){:new_window}.
1. Create a new preproduction Satellite location in a non-default resource group. For more information on setting up a satellite location, refer to [Setting up Satellite locations](https://cloud.ibm.com/docs/satellite?topic=satellite-locations) in the {{site.data.keyword.satellitelong}} documentation.
1. When creating Satellite location, choose `Manual setup`.

![Choose manual setup from Setup card options](images/manual-setup.png)

1. Complete the form to create your Satellite location:
- Provide a name
- Resource group
- Zones 1, 2, and 3 should retain their default values: zone-1, zone-2, zone-3
1. Click `Done editing` to complete your Satellite creation.

![Fill out form to create satellite location](images/satellite-location.png)


<!-- Introduce each major step with a description of what it will accomplish. If there are sequential substeps, use an ordered list for each substep. Don't include the step number. -->

_If you have substeps, introduce them and then follow with the steps as paragraph chunks. Do not use numbers or substep labels for a single substep or unless it is a true sequence._

_For commands, introduce the command and then surround what the user must enter in the command prompt with three backticks, set the programming language if it applies, and follow with a pre attribute if you want the copy button applied._ For example:
"Now you're ready to start working with the app. First, clone the repo with the sample app code.
  ```sh
  git clone https://github.com/IBM-Cloud/get-started-node
  ```
  {: pre}

  Then, change the directory to where the sample app is located.

  ```sh
  cd get-started-node
  ```
  {: pre}

## _Title should be task oriented and descriptive_
{: #anchor_value}
{: step}

## _Title should be task oriented and descriptive_
{: #anchor_value}
{: step}

## _Title should be task oriented and descriptive_
{: #anchor_value}
{: step}

_If you have any "tips" to include for this step, add the content in the flow where you would like it to appear. This information should be not be required information for completing the step, but helpful information in explaining additional concepts about what the user is doing in the step. It should be in the following format:_

One to two sentences of content that can include inline links or lists.
{: tip}

_You can have multiple "tips" per step. Each tip will output with a *Tip:* label and be formatted in a nested, styled box._

## Next steps
{: #anchor_value}

_What's the single thing the user needs to do next? Think "guided journey." Either provide information that leads the user to production use, for example HA, how to make a service secure, or how to connect to on-premise data. Or you can point the user to another tutorial. Give a choice between two options max._
