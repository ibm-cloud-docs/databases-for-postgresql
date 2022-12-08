---

copyright:
  years: 2021
lastupdated: "2022-03-31"

keywords: IBM Cloud, databases, Satellite, ICD, IAM authorization

subcollection: databases-for-postgresql

---

{:external: .external target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{{site.data.keyword.attribute-definition-list}}

# Managing IAM access for {{site.data.keyword.databases-for-postgresql_full}}
{: #manage-iam-postgresql}

Access to `databases-for-postgresql` service instances for users in your account is controlled by {{site.data.keyword.cloud}} Identity and Access Management (IAM). Every user that accesses the `databases-for-postgresql` service in your account must be assigned an access policy with an IAM role. Review the following roles, actions, and more to help determine the best way to assign access to `databases-for-postgresql`. 
{: shortdesc}

The access policy that you assign users in your account determines what actions a user can perform within the context of the service or specific instance that you select. The allowable actions are customized and defined by the `databases-for-postgresql` as operations that are allowed to be performed on the service. Each action is mapped to an IAM platform or service role that you can assign to a user.

If a specific role and its actions don't fit the use case that you're looking to address, you can [create a custom role](/docs/account?topic=account-custom-roles#custom-access-roles) and pick the actions to include.
{: tip}

IAM access policies enable access to be granted at different levels. Some of the options include the following: 

* Access across all instances of the service in your account
* Access to an individual service instance in your account <!-- if this applies -->
* Access to a specific resource within an instance, _such as resource type `bucket`_ <!-- if this applies list what resoureceType attributes are supported, for example COS buckets or Kubernetes namespaces -->

Review the following tables that outline what types of tasks each role allows for when you're working with the `databases-for-postgresql` service. Platform management roles enable users to perform tasks on service resources at the platform level, for example, assign user access to the service, create or delete instances, and bind instances to applications. Service access roles enable users access to `databases-for-postgresql` and the ability to call the `databases-for-postgresql's` API. For information about the exact actions that are mapped to each role, see [`databases-for-postgresql`](_YourSubHeadingLink_).
<!-- IMPORTANT: This link should go directly to your service's heading in https://cloud.ibm.com/docs/account?topic=account-iam-service-roles-actions, for example [`service-name`](/docs/account?topic=account-iam-service-roles-actions#certificate-manager) -->

<!-- This is a high level view of what the platform roles allow users to do. Use a plain language description about what kind of tasks can be completed or the common jobs to be done that users can expect to accomplish when having each role assigned. -->

<!-- Include any service-specific custom roles that your service has registered.
To find the role_id values, run the `ibmcloud iam roles` command or go to the Manage>Access (IAM)>Roles console page. Select your service, then use the List of Actions icon for the row of the role that you want to get the ID value for, and click Details. It is part of the CRN. For example, in crn:v1:bluemix:public:cloud-object-storage::::serviceRole:ContentReader, ContentReader is the ID value. -->

| Platform role |  Description of actions | 
|---------------|-------------------------|
| Viewer                 |  As a viewer, you can view database instances but you can't make configuration changes. |
| Operator               |  As an operator, you can view database instances and make configuration changes including managing database credentials.	            |
| Editor                 |  As an editor, you can perform all platform actions (including making configuration changes and managing credentials) except for managing the account and assigning access policies.	            | 
| Administrator          |  As an administrator, you can perform all platform actions including assigning access policies to other users.	            |
{: row-headers}
{: class="simple-tab-table"}
{: caption="Table 1. IAM platform roles" caption-side="bottom"}
{: #iamrolesplatform}
{: tab-title="Platform roles"}
{: tab-group="IAM"}

## Assigning access to databases-for-postgresql in the console
{: #assign-access-console}
{: ui}

There are two common ways to assign access in the console:

* Access policies per user. You can manage access policies per user from the **Manage** > **Access (IAM)** > **Users** page in the console. For information about the steps to assign IAM access, see [Managing access to resources](https://cloud.ibm.com/docs/account?topic=account-assign-access-resources&interface=ui#access-resources-console).
* Access groups. Access groups are used to streamline access management by assigning access to a group once, then you can add or remove users as needed from the group to control their access. You manage access groups and their access from the **Manage** > **Access (IAM)** > **Access groups** page in the console. For more information, see [Assigning access to a group in the console](/docs/account?topic=account-groups&interface=ui#access_ag).


## Assigning access to {{site.data.keyword.databases-for-postgresql_full}} in the CLI
{: #assign-access-cli}
{: cli}

For step-by-step instructions for assigning, removing, and reviewing access, see [Assigning access ro resources by using the CLI](/docs/account?topic=account-assign-access-resources&interface=cli#access-resources-cli). 


If you manage your service through the IBM Cloud CLI and the cloud databases plug-in, you can create a new user with `cdb user-create`. For example, to create a new user for an "example-deployment", use the following command.

```bash
ibmcloud cdb user-create example-deployment <newusername> <newpassword>
```
{: pre}

Once the task has finished, you can retrieve the new user's connection strings with the `ibmcloud cdb deployment-connections` command.

## Assigning access to {{site.data.keyword.databases-for-postgresql_full}} by using the API
{: #assign-access-api}
{: api}

For step-by-step instructions for assigning, removing, and reviewing access, see [Assigning access to resources by using the API](/docs/account?topic=account-assign-access-resources&interface=api) or the [Create a policy API docs](/apidocs/iam-policy-management#create-policy). Role cloud resource names (CRN) in the following table are used to assign access with the API.

The _Foundation Endpoint_ that is shown on the _Overview_ panel _Deployment details_ of your service provides the base URL to access this deployment through the API. To create and manage users, use the base URL with the `/users` endpoint, like the following example.

```bash
curl -X POST 'https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users' -H "Authorization: Bearer $APIKEY" -H "Content-Type: application/json" -d '{"username":"jane_smith", "password":"newsupersecurepassword"}'
```
{: pre}

Once the task has finished, you can retrieve the new user's connection strings, from the `/users/{userid}/connections` endpoint.

| Role name | Role CRN | 
|---------------|-----------------|
| Viewer                 | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Viewer`        |
| Operator               | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Operator`      | 
| Editor                 | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Editor`        | 
| Administrator          | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Administrator` | 
{: caption="Table 2. Role ID values for API use" caption-side="bottom"}

## Actions for {{site.data.keyword.databases-for}} API
{: #actions}

Access to certain API endpoints and requests is governed by role. The following lists the access policy for each role for {{site.data.keyword.cloud}} Databases. 

### Viewer 
{: #viewer}

The allowed actions for the Viewer role.
```bash
GET /v4/ibm/deployables
Read Deployables
---
GET /v4/ibm/regions
Read Discover available regions
---
GET /v4/ibm/tasks/:task_id
Read a Task
---
GET /v4/ibm/backups/:backup_id
Read a Backup
---
GET /v4/ibm/deployments/:deployment_id
Read a Deployment
---
GET /v4/ibm/deployables/:deployable_id/groups
Read deployable group
---
GET /v4/ibm/deployments/:deployment_id/point_in_time_recovery_data
Read all deployment point-in-time-recovery data
---
GET /v4/ibm/deployments/:deployment_id/tasks
Read all deployment tasks
---
GET /v4/ibm/deployments/:deployment_id/backups
Read all deployment backups
---
GET /v4/ibm/deployments/:deployment_id/remotes
Read all deployment remotes
---
GET /v4/ibm/deployments/:deployment_id/groups
Read all deployment groups
---
GET /v4/ibm/deployments/:deployment_id/configuration/schema
Read deployment configuration schema
---
GET /v4/ibm/deployments/:deployment_id/users/:user_id
Read a DeploymentUser
---
GET /v4/ibm/deployments/:deployment_id/users/:user_id/connections
Read deployment user connections
---
GET /v4/ibm/deployments/:deployment_id/users/:user_id/connections/:endpoint_type
Read deployment user connections
---
POST /v4/ibm/deployments/:deployment_id/users/:user_id/connections
Create deployment user connections
---
POST /v4/ibm/deployments/:deployment_id/users/:user_id/connections/:endpoint_type
Create deployment user connections
---
GET /v4/ibm/deployments/:deployment_id/whitelists/ip_addresses
Read Allowlisted IP Addresses
```

### Operator and Editor
{: #operator}

The Operator and Editor roles are functionally the same for {{site.data.keyword.databases-for}}. This list contains allowed actions for the Operator and the Editor roles.
```bash
GET /v4/ibm/deployables
Read Deployables
---
GET /v4/ibm/regions
Read Discover available regions
---
GET /v4/ibm/tasks/:task_id
Read a Task
---
GET /v4/ibm/backups/:backup_id
Read a Backup
---
GET /v4/ibm/deployments/:deployment_id
Read a Deployment
---
PATCH /v4/ibm/deployments/:deployment_id
Update a Deployment
---
GET /v4/ibm/deployables/:deployable_id/groups
Read deployable group
---
GET /v4/ibm/deployments/:deployment_id/point_in_time_recovery_data
Read all deployment point-in-time-recovery data
---
GET /v4/ibm/deployments/:deployment_id/tasks
Read all deployment tasks
---
GET /v4/ibm/deployments/:deployment_id/backups
Read all deployment backups
---
POST /v4/ibm/deployments/:deployment_id/backups
Create an on-demand backup
---
GET /v4/ibm/deployments/:deployment_id/remotes
Read all deployment remotes
---
PATCH /v4/ibm/deployments/:deployment_id/remotes
Update a remote replica
---
POST /v4/ibm/deployments/:deployment_id/remotes/resync
Resync remote replica
---
GET /v4/ibm/deployments/:deployment_id/groups
Read all deployment groups
---
PATCH /v4/ibm/deployments/:deployment_id/groups/:group_id
Read deployment group
---
DELETE /v4/ibm/deployments/:deployment_id/management/database_connections
Kill all database connections
---
PATCH /v4/ibm/deployments/:deployment_id/configuration
Update deployment configuration
---
GET /v4/ibm/deployments/:deployment_id/configuration/schema
Read deployment configuration schema
---
POST /v4/ibm/deployments/:deployment_id/users
Create a DeploymentUser
---
GET /v4/ibm/deployments/:deployment_id/users/:user_id
Read a DeploymentUser
---
PATCH /v4/ibm/deployments/:deployment_id/users/:user_id
Update a DeploymentUser
---
DELETE /v4/ibm/deployments/:deployment_id/users/:user_id
Remove a DeploymentUser
---
GET /v4/ibm/deployments/:deployment_id/users/:user_id/connections
Read deployment user connections
---
GET /v4/ibm/deployments/:deployment_id/users/:user_id/connections/:endpoint_type
Read deployment user connections
---
POST /v4/ibm/deployments/:deployment_id/users/:user_id/connections
Create deployment user connections
---
POST /v4/ibm/deployments/:deployment_id/users/:user_id/connections/:endpoint_type
Create deployment user connections
---
GET /v4/ibm/deployments/:deployment_id/whitelists/ip_addresses
Read Allowlisted IP Addresses
---
POST /v4/ibm/deployments/:deployment_id/whitelists/ip_addresses
Create an Allowlisted IP Addresses
---
DELETE /v4/ibm/deployments/:deployment_id/whitelists/ip_addresses/:ip_address_id
Remove an Allowlisted IP Addresses
---
PUT /v4/ibm/deployments/:deployment_id/whitelists/ip_addresses
Bulk allowlist IP addresses
---
POST /v4/ibm/deployments/:deployment_id/elasticsearch/file_syncs
Create elasticsearch file sync
```

### Administrator
{: #admin}

The allowed actions for the Administrator role.
```bash
GET /v4/ibm/deployables
Read Deployables
---
GET /v4/ibm/regions
Read Discover available regions
---
GET /v4/ibm/tasks/:task_id
Read a Task
---
GET /v4/ibm/backups/:backup_id
Read a Backup
---
GET /v4/ibm/deployments/:deployment_id
Read a Deployment
---
PATCH /v4/ibm/deployments/:deployment_id
Update a Deployment
---
GET /v4/ibm/deployables/:deployable_id/groups
Read deployable group
---
GET /v4/ibm/deployments/:deployment_id/point_in_time_recovery_data
Read all deployment point-in-time-recovery data
---
GET /v4/ibm/deployments/:deployment_id/tasks
Read all deployment tasks
---
GET /v4/ibm/deployments/:deployment_id/backups
Read all deployment backups
---
POST /v4/ibm/deployments/:deployment_id/backups
Create an on-demand backup
---
GET /v4/ibm/deployments/:deployment_id/remotes
Read all deployment remotes
---
PATCH /v4/ibm/deployments/:deployment_id/remotes
Update a remote replica
---
POST /v4/ibm/deployments/:deployment_id/remotes/resync
Resync remote replica
---
GET /v4/ibm/deployments/:deployment_id/groups
Read all deployment groups
---
PATCH /v4/ibm/deployments/:deployment_id/groups/:group_id
Read deployment group
---
DELETE /v4/ibm/deployments/:deployment_id/management/database_connections
Kill all database connections
---
PATCH /v4/ibm/deployments/:deployment_id/configuration
Update deployment configuration
---
GET /v4/ibm/deployments/:deployment_id/configuration/schema
Read deployment configuration schema
---
POST /v4/ibm/deployments/:deployment_id/users
Create a DeploymentUser
---
GET /v4/ibm/deployments/:deployment_id/users/:user_id
Read a DeploymentUser
---
PATCH /v4/ibm/deployments/:deployment_id/users/:user_id
Update a DeploymentUser
---
DELETE /v4/ibm/deployments/:deployment_id/users/:user_id
Remove a DeploymentUser
---
GET /v4/ibm/deployments/:deployment_id/users/:user_id/connections
Read deployment user connections
---
GET /v4/ibm/deployments/:deployment_id/users/:user_id/connections/:endpoint_type
Read deployment user connections
---
POST /v4/ibm/deployments/:deployment_id/users/:user_id/connections
Create deployment user connections
---
POST /v4/ibm/deployments/:deployment_id/users/:user_id/connections/:endpoint_type
Create deployment user connections
---
GET /v4/ibm/deployments/:deployment_id/whitelists/ip_addresses
Read Allowlisted IP Addresses
---
POST /v4/ibm/deployments/:deployment_id/whitelists/ip_addresses
Create an Allowlisted IP Addresses
---
DELETE /v4/ibm/deployments/:deployment_id/whitelists/ip_addresses/:ip_address_id
Remove an Allowlisted IP Addresses
---
PUT /v4/ibm/deployments/:deployment_id/whitelists/ip_addresses
Bulk allowlist IP addresses
---
POST /v4/ibm/deployments/:deployment_id/elasticsearch/file_syncs
Create elasticsearch file sync
```

## Adding users to _Service Credentials_
{: #user-management-adding-users-service-cred}

Creating a new user from the CLI doesn't automatically populate that user's connection strings into _Service Credentials_. If you want to add them there, you can create a new credential with the existing user information.

Enter the user name and password in the JSON field _Add Inline Configuration Parameters_, or specify a file where the JSON information is stored. For example, putting `{"existing_credentials":{"username":"Robert","password":"supersecure"}}` in the field generates _Service Credentials_ with the username "Robert" and password "supersecure" filled into connection strings.

Generating credentials from an existing user does not check for or create that user.
{: tip}

<!-- Tailor this example to your service --> 

The following example is for assigning the `<Object Writer>` role for `<Cloud Object Storage>`:

Use `<programmatic_service_name>` for the service name, and refer to the Role ID values table to ensure that you're using the correct value for the CRN.
{: tip}
<!--The `<programmatic_service_name` in the note above is important to include because the service name in the UI often doesn't match the service name that should be used to make an API call.-->

## Assigning access to databases-for-postgresql by using Terraform
{: #assign-access-terraform}
{: terraform}

<!-- Tailor this example to your service --> 

The following example is for assigning the `<Object Writer>` role for `<Cloud Object Storage>`:

Use `<programmatic_service_name>` for the service name.
{: tip}
<!--The `<programmatic_service_name` in the note above is important to include because the service name in the UI often doesn't match the service name that should be used when assigning access using Terraform.-->

```terraform
resource "ibm_iam_user_policy" "policy" {
  ibm_id = "test@example.com"
  roles  = ["Object Writer"]

  resources {
    service = "cloud-object-storage"
  }
}
```
{: codeblock}

For more information, see [ibm_iam_user_policy](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs/resources/iam_user_policy){: external}.
