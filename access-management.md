---

copyright:
  years: 2018
lastupdated: "2018-08-14"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}

# Managing user access

Access to {{site.data.keyword.databases-for-postgresql_full}} service instances for users in your account is controlled by {{site.data.keyword.cloud_notm}} Identity and Access Management (IAM). Every user that accesses the {{site.data.keyword.databases-for-postgresql}} service in your account must be assigned an access policy with an IAM user role defined. That policy determines what actions the user can perform within the context of the service or instance you select. The allowable actions are customized and defined by the {{site.data.keyword.cloud_notm}} service as operations that are allowed to be performed on the service. The actions are then mapped to IAM user roles.

Policies enable access to be granted at different levels. Some of the options include: 
* Access across all instances of the service in your account
* Access to an individual service instance in your account
* Access to a specific resource within an instance
* Access to all IAM-enabled services in your account

After you define the scope of the access policy, you assign a role. Review the following tables that outline what actions each role allows within the {{site.data.keyword.databases-for-postgresql}} service.

For more information about assigning user roles in {{site.data.keyword.cloud_notm}}, see [Managing IAM access](https://{DomainName}net/docs/iam/iamusermanage.html#iamusermanage).

The following table details actions that are mapped to service management roles. Service management roles enable users to perform tasks on service resources at the service level, for example assign user access for the service, create or delete service IDs, create instances, and bind instances to applications.

Service management role | Description of actions | Example actions
-----------------|-----------------|-----------------
Viewer | As a viewer, you can view service instances, but you can't modify them. | View Service Overview and View Alerts
Editor | As an editor, you can perform all platform actions except for managing the account and assigning access policies. | Scale a Deployment and Change a Deployment's Password
Operator | N/A | N/A
Administrator | As an administrator, you can perform all platform actions based on the resource this role is being assigned, including assigning access policies to other users. | Scale a Deployment, Change a Deployment's Password, and Assign Access Policies
{: caption="Table 1. IAM user roles and actions" caption-side="top"}

## Actions for {{site.data.keyword.databases-for-postgresql}} API

Platform Action| Operation on service| Role
-------------|--------------------|--------------
GET /v4/ibm/deployables| Read Deployables| Administrator, Editor, Viewer 
GET /v4/ibm/tasks/:task_id| Read a Task| Administrator, Editor, Viewer
GET /v4/ibm/backups/:backup_id | Read a Backup| Administrator, Editor, Viewer
POST /v4/ibm/deployments | Create a Deployment | Administrator, Editor
GET /v4/ibm/deployments/:deployment_id | Read a Deployment | Administrator, Editor, Viewer
DELETE /v4/ibm/deployments/:deployment_id | Remove a Deployment | Administrator, Editor 
GET /v4/ibm/deployables/:deployable_id/groups | Read deployable group | Administrator, Editor, Viewer
GET /v4/ibm/deployments/:deployment_id/tasks| Read all deployment tasks | Administrator, Editor, Viewer 
GET /v4/ibm/deployments/:deployment_id/backups | Read all deployment backups | Administrator, Editor, Viewer
POST /v4/ibm/deployments/:deployment_id/backups | Create an on-demand backup | Administrator, Editor
GET /v4/ibm/deployments/:deployment_id/groups | Read all deployment groups | Administrator, Editor, Viewer
PATCH /v4/ibm/deployments/:deployment_id/groups/:group_id | Read deployment group | Administrator, Editor
POST /v4/ibm/deployments/:deployment_id/users | Create a Deployment User | Administrator, Editor
PATCH /v4/ibm/deployments/:deployment_id/users/:user_id | Update a Deployment User | Administrator, Editor
DELETE /v4/ibm/deployments/:deployment_id/users/:user_id | Remove a Deployment User | Administrator, Editor
GET /v4/ibm/deployments/:deployment_id/users/:user_id/connections | Read deployment user connections | Administrator, Editor, Viewer
POST /v4/ibm/deployments/:deployment_id/users/:user_id/connections | Create deployment user connections | Administrator, Editor
{: caption="Table 2. Platform actions and operations" caption-side="top"}

