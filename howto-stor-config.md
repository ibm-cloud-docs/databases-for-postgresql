---

copyright:
  years: 2021
lastupdated: "2021-06-16"

keywords: IBM Cloud, databases, Satellite, ICD, storage configuration

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
{:java: .ph data-hd-programlang='java'}
{:javascript: .ph data-hd-programlang='javascript'}
{:swift: .ph data-hd-programlang='swift'}
{:curl: .ph data-hd-programlang='curl'}
{:video: .video}
{:step: data-tutorial-type='step'}
{:tutorial: data-hd-content-type='tutorial'}
{:ui: .ph data-hd-interface='ui'}
{:cli: .ph data-hd-interface='cli'}
{:api: .ph data-hd-interface='api'}

{:shortdesc: .shortdesc}
{:screen: .screen}  
{:codeblock: .codeblock}  
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:external: target="_blank" .external}
<!-- {:step: data-tutorial-type='step'} <!-- Apply to steps for automatic numbering -->

<!-- The title of your tutorial should be in active voice and and start with a verb. If you include product names, makes sure to use the non-trademarked short version conref. -->
<!-- Make sure each H1/H2/H3/etc. heading is _unique_ to your tutorial by adding a short but human-readable identifier. For example, instead of just "#overview", use "#cd-kube-overview" -->

# Creating Storage Configuration
{: #postgres-sat-prereqs}

To set up a block storage configuration, you will need to invoke a CLI command.
{: shortdesc}

## Invoke CLI command
{: #cd-postgresql-clicommand}

Here is a sample AWS command:

```
 ic sat storage config create  \
  --name 'aws-ebs-config-storage-testing-1' \
  --template-name 'aws-ebs-csi-driver' \
  --template-version '0.9.14' \
  --location 'c2tace1w07k5gvilnqq0' \
  -p "aws-access-key=${SAT_EBS_ADMIN_KEY_ID}" \
  -p "aws-secret-access-key=${SAT_EBS_ADMIN_KEY}"
  ```
that will output the following:

```
Storage configuration 'aws-ebs-config-storage-testing-1' was successfully created with ID 'b1005956-2d9d-4d46-9fbf-0589c9437716'.
```

Here is a sample NetApp command:

```
$ ibmcloud sat storage config create --source-org 'ntap-troy' --source-branch 'develop' --location '<location-id>' --name 'netapp-trident'  --template-name 'netapp-trident' --template-version '21.04'
$ ibmcloud sat storage config create --source-org 'ntap-troy' --source-branch 'develop' --location '<location-id>' --name 'netapp-san'  --template-name 'netapp-ontap-san' --template-version '21.04' --param "dataLIF=169.60.138.166" --param "managementLIF=169.60.138.164" --param  "svm=<svm name>" --param "username=admin" --param 'password=<password>'
$ ibmcloud sat storage assignment create --name "trident-assignment" --cluster '<cluster-id>' --config 'netapp-trident'
$ ibmcloud sat storage assignment create --name "san-assignment" --cluster '<cluster-id>' --config 'netapp-san'
```
