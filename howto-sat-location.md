---

copyright:
  years: 2021
lastupdated: "2021-06-29"

keywords: IBM Cloud, databases, Satellite, ICD, get started

subcollection: databases-for-postgresql

content-type: tutorial
services: databases-for-postgresql
completion-time: 15m

---

{:codeblock: .codeblock}
{:screen: .screen}
{:download: .download}
{:external: target="_blank" .external}
{:faq: data-hd-content-type='faq'}
{:gif: data-image-type='gif'}
{:important: .important}
{:note: .note}
{:pre: .pre}
{:tip: .tip}
{:preview: .preview}
{:deprecated: .deprecated}
{:beta: .beta}
{:term: .term}
{:shortdesc: .shortdesc}
{:script: data-hd-video='script'}
{:support: data-reuse='support'}
{:table: .aria-labeledby="caption"}
{:troubleshoot: data-hd-content-type='troubleshoot'}
{:help: data-hd-content-type='help'}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}
{:tsSymptoms: .tsSymptoms}
{:curl: .ph data-hd-programlang='curl'}
{:step: data-tutorial-type='step'}
{:tutorial: data-hd-content-type='tutorial'}
{:ui: .ph data-hd-interface='ui'}
{:cli: .ph data-hd-interface='cli'}
{:api: .ph data-hd-interface='api'}

# IBM Cloud™ Databases for PostgreSQL enabled by IBM Cloud Satellite
{: #postgresql-satellite}

With IBM Cloud™ Databases (ICD) for PostgreSQL enabled by IBM Cloud Satellite, you can deploy IBM Cloud Database instances into a Satellite location. The IBM Cloud™ Database (ICD) service will then install an ICD Satellite service cluster in your Satellite location upon which your database instances will be deployed.
IBM Cloud™ Databases for PostgreSQL enabled by IBM Cloud Satellite supports Satellite locations on [Amazon Web Services (AWS)](https://cloud.ibm.com/docs/satellite?topic=satellite-aws), as well as on your on-premises data centers that are attached to the IBM Cloud management location in Washington, DC.
Before proceeding, you should refer to the [Satellite usage requirements](/docs/satellite?topic=satellite-requirements).
{: shortdesc}

## Step 1: Create your Satellite location
{: #postgresql-satellite-location}

### Before you begin:
Follow the steps to setup a Satellite location [here](/docs/satellite?topic=satellite-locations) in detail. We recommend creating a new [Satellite location](/docs/satellite?topic=satellite-locations) by selecting the **Manual setup** tab:
- For the management location, choose **Washington DC**.
- If you create your Satellite location on AWS, adjust the **host zones** to AWS-default zone names, for example: **us-east-1a**, **us-east-1b**, **us-east-1c**.

Before proceeding with **Step 2**, you should have set up your Satellite location properly and ensured the Satellite control plane is up and running.

## Step 2: Prepare Satellite location for IBM Cloud™ Databases
{: #postgresql-satellite-host}

Before deploying the IBM Cloud™ Databases for PostgreSQL enabled by IBM Cloud Satellite service, you should prepare your Satellite location.

### Attach additional hosts to the Satellite location

These additional attached worker nodes are used to create a service cluster upon which the database instances will later be deployed.
Attach to your Satellite location:
- three type **8x32** hosts
   - on AWS, choose three hosts of type **AWS m5d.2xlarge**
- three type **32*128** hosts
   - on AWS choose three hosts of type **AWS m5d.8xlarge**

### Create a Satellite block storage configuration

IBM Cloud™ Databases for PostgreSQL enabled by IBM Cloud Satellite supports the following [Satellite storage templates](https://cloud.ibm.com/docs/satellite?topic=satellite-sat-storage-template-ov):
- [Amazon Elastic Block Storage (EBS)](/docs/satellite?topic=satellite-config-storage-ebs) storage template for a Satellite location on AWS
- [NetApp ONTAP-SAN](/docs/satellite?topic=satellite-config-storage-netapp) storage template for a Satellite location within your datacenter (on-premises)

To create an AWS Satellite location, see [Creating an AWS EBS storage configuration](/docs/satellite?topic=satellite-config-storage-ebs).

Copy the following the command and replace the variables with the parameters for your storage configuration.

```
ibmcloud sat storage config create  \
  --name 'aws-ebs-config-storage-testing-1' \
  --template-name 'aws-ebs-csi-driver' \
  --template-version '0.9.14' \
  --location 'c2tace1w07k5gvilnqq0' \
  -p "aws-access-key=${SAT_EBS_ADMIN_KEY_ID}" \
  -p "aws-secret-access-key=${SAT_EBS_ADMIN_KEY}"
```

For a Satellite location within your on-prem data center, see [NetApp ONTAP-SAN](/docs/satellite?topic=satellite-config-storage-netapp).

Copy the following the command and replace the variables with the parameters for your storage configuration.

```
ibmcloud sat storage config create \
    --name 'netapp-san'  \
    --location '<location-id>' \
    --template-name 'netapp-ontap-san' \
    --template-version '21.04'  \
    --param "dataLIF=<dataLIF IP address>" \
    --param "managementLIF=<dataLIF IP address" \
    --param  "svm=<svm name>" \
    --param "username=admin" \
    --param 'password=<password>' \
    --source-org 'ntap-org'  \
    --source-branch 'develop'
````

### Configure IAM Authorizations
{: #cd-postgresql-iamauth}

- Configure your IAM Authorizations under the **Manage** tab.
- Choose the **Authorizations** tab from the lefthand menu.
- Click the **create** button to create an authorization to allow a service instance access to another service instance.

## Step 3: Grant a service authorization
{: #cd-postgresql-serviceauth}

The source service is the service that is granted access to the target service. The roles you select define the level of access for this service. The target service is the service you are granting permission to be accessed by the source service based on the assigned roles.

- In the **Source Service** field, select **Databases for PostgreSQL development**.
- In the **Target Service** field, select **Satellite**.
- Select all options:
  - **Satellite Cluster Creator**
  - **Satellite Link Administrator**
  - **Satellite Link Source Access Controller**
 - Then **Authorize**.

## Step 4: Deploy IBM Cloud™ Databases for PostgreSQL enabled by IBM Cloud Satellite

When you create a new database service instance, a service cluster will first be deployed into your Satellite location. This service cluster step requires that you have prepared the Satellite location properly (see Step 2 above). Deployment of the service cluster can take up to one hour. When the service cluster is created you have to create a storage assignment manually **before** the database instance will be started.

## Step 5: Create a Storage Assignment

When the service cluster is available in your Satellite location, you have to create a Satellite storage assignment. This will allow the service cluster to create volumes on the previously configured storage.

See the example below for an AWS Satellite location storage assignment:
```
ibmcloud sat storage assignment create  \
    --name "ebs-assignment"  \
    --service-cluster-id <ROKS-Service-cluster-ID>  \
    --config 'aws-ebs-config-storage-testing-1'
```

See the example below for an NetApp ONTAP-SAN on-prem Satellite location storage assignment:

```
ibmcloud sat storage assignment create \
  --name "san-assignment"  \
  --service-cluster-id <ROKS-Service-cluster-ID>  \
  --config 'netapp-san'
```

After storage assignment has been created, allow up to 30 minutes for the database instance to be ready for usage.
