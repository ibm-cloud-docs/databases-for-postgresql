---

copyright:
  years:  2017, 2022
lastupdated: "2022-03-28"

keywords: IAM access for _servicename_, permissions for _servicename_, identity and access management for _servicename_, roles for _servicename_, actions for _servicename_, assigning access for _servicename_

subcollection: _your-subcollection_

---

{{site.data.keyword.attribute-definition-list}}

_Include this file in the **How to** nav group of your toc.yaml file_

# Managing IAM access for _servicename_
{: #iam-docs-template}

Access to `<service_name>` service instances for users in your account is controlled by {{site.data.keyword.cloud}} Identity and Access Management (IAM). Every user that accesses the `<service_name>` service in your account must be assigned an access policy with an IAM role. Review the following roles, actions, and more to help determine the best way to assign access to `<service_name>`. 
{: shortdesc}

The access policy that you assign users in your account determines what actions a user can perform within the context of the service or specific instance that you select. The allowable actions are customized and defined by the `<service_name>` as operations that are allowed to be performed on the service. Each actions is mapped to an IAM platform or service role that you can assign to a user.

If a specific role and its actions don't fit the use case that you're looking to address, you can [create a custom role](/docs/account?topic=account-custom-roles#custom-access-roles) and pick the actions to include.
{: tip}

IAM access policies enable access to be granted at different levels. Some of the options include the following: 

* Access across all instances of the service in your account
* Access to an individual service instance in your account <!-- if this applies -->
* Access to a specific resource within an instance, _such as resource type `bucket`_ <!-- if this applies list what resoureceType attributes are supported, for example COS buckets or Kubernetes namespaces -->

Review the following tables that outline what types of tasks each role allows for when you're working with the `<service_name>` service. Platform management roles enable users to-perform tasks on service resources at the platform level, for example, assign user access to the service, create or delete instances, and bind instances to applications. Service access roles enable users access to `<service_name>` and the ability to call the `<service_name>'s` API. For information about the exact actions that are mapped to each role, see [`<service_name>`](_YourSubHeadingLink_).
<!-- IMPORTANT: This link should go directly to your service's heading in https://cloud.ibm.com/docs/account?topic=account-iam-service-roles-actions, for example [`service-name`](/docs/account?topic=account-iam-service-roles-actions#certificate-manager) -->

<!-- This is a high level view of what the platform roles allow users to do. Use a plain language description about what kind of tasks can be completed or the common jobs to be done that users can expect to accomplish when having each role assigned. -->

<!-- Include any service-specific custom roles that your service has registered.
To find the role_id values, run the `ibmcloud iam roles` command or go to the Manage>Access (IAM)>Roles console page. Select your service, then use the List of Actions icon for the row of the role that you want to get the ID value for, and click Details. It is part of the CRN. For example, in crn:v1:bluemix:public:cloud-object-storage::::serviceRole:ContentReader, ContentReader is the ID value. -->

| Platform role |  Description of actions | 
|---------------|-------------------------|
| Viewer                 |  Description of what users can accomplish; think tasks. |
| Operator               |  Description            |
| Editor                 |  Description            | 
| Administrator          |  Description            |
{: row-headers}
{: class="simple-tab-table"}
{: caption="Table 1. IAM platform roles" caption-side="bottom"}
{: #iamrolesplatform}
{: tab-title="Platform roles"}
{: tab-group="IAM"}

| Service role |  Description of actions | 
|--------------|------------------------|
| Reader         | Description of what users can accomplish; think tasks.   | 
| Writer         | Description            |
| Manager        | Description            | 
| Content Reader | Description            |
{: row-headers}
{: class="simple-tab-table"}
{: caption="Table 1. IAM service access roles" caption-side="bottom"}
{: #iamrolesservice}
{: tab-title="Service roles"}
{: tab-group="IAM"}

## Assigning access to <service_name> in the console
{: #assign-access-console}
{: ui}

There are two common ways to assign access in the console:

* Access policies per user. You can manage access policies per user from the **Manage** > **Access (IAM)** > **Users** page in the console. For information about the steps to assign IAM access, see [Managing access to resources](https://cloud.ibm.com/docs/account?topic=account-assign-access-resources&interface=ui#access-resources-console).
* Access groups. Access groups are used to streamline access management by assigning access to a group once, then you can add or remove users as needed from the group to control their access. You manage access groups and their access from the **Manage** > **Access (IAM)** > **Access groups** page in the console. For more information, see [Assigning access to a group in the console](/docs/account?topic=account-groups&interface=ui#access_ag).


## Assigning access to <service_name> in the CLI
{: #assign-access-cli}
{: cli}

For step-by-step instructions for assigning, removing, and reviewing access, see [Assigning access ro resources by using the CLI](/docs/account?topic=account-assign-access-resources&interface=cli#access-resources-cli). The following example shows a command for assigning the `<Object Writer>` role for `<Cloud Object Storage>`:

Use `<programmatic_service_name>` for the service name. Also, use quotations around role names that are more than one word like the example here.
{: tip}
<!--The `<programmatic_service_name` in the note above is important to include because the service name in the UI often doesn't match the service name that should be used to make an API call or run a CLI command.-->

<!-- Tailor this example to your service --> 

```bash
ibmcloud iam user-policy-create USER@EXAMPLE.COM --service-name cloud-object-storage --roles "Object Writer"
```
{: pre}

## Assigning access to <service_name> by using the API
{: #assign-access-api}
{: api}

For step-by-step instructions for assigning, removing, and reviewing access, see [Assigning access to resources by using the API](/docs/account?topic=account-assign-access-resources&interface=api) or the [Create a policy API docs](/apidocs/iam-policy-management#create-policy). Role cloud resource names (CRN) in the following table are used to assign access with the API.


| Role name | Role CRN | 
|---------------|-----------------|
| Viewer                 | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Viewer`        |
| Operator               | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Operator`      | 
| Editor                 | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Editor`        | 
| Administrator          | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Administrator` | 
| Reader         | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Reader`        |
| Writer         | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Writer`        | 
| Manager        | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:Manager`       | 
| Object Writer | `crn:v1:bluemix:public:cloud-object-storage::::serviceRole:ObjectWriter` | 
{: caption="Table 2. Role ID values for API use" caption-side="bottom"}

<!-- Tailor this example to your service --> 

The following example is for assigning the `<Object Writer>` role for `<Cloud Object Storage>`:

Use `<programmatic_service_name>` for the service name, and refer to the Role ID values table to ensure that you're using the correct value for the CRN.
{: tip}
<!--The `<programmatic_service_name` in the note above is important to include because the service name in the UI often doesn't match the service name that should be used to make an API call.-->

```curl 
curl -X POST 'https://iam.cloud.ibm.com/v1/policies' -H 'Authorization: Bearer $TOKEN' -H 'Content-Type: application/json' -d '{
  "type": "access",
  "description": "Object Writer role for Cloud Object Storage",
  "subjects": [
    {
      "attributes": [
        {
          "name": "iam_id",
          "value": "IBMid-123453user"
        }
      ]
    }'
  ],
  "roles":[
    {
      "role_id": "crn:v1:bluemix:public:cloud-object-storage::::serviceRole:ObjectWriter"
    }
  ],
  "resources":[
    {
      "attributes": [
        {
          "name": "accountId",
          "value": "$ACCOUNT_ID"
        },
        {
          "name": "serviceName",
          "value": "cloud-object-storage"
        }
      ]
    }
  ]
}
```
{: curl}
{: codeblock}

```java
SubjectAttribute subjectAttribute = new SubjectAttribute.Builder()
      .name("iam_id")
      .value("IBMid-123453user")
      .build();

PolicySubject policySubjects = new PolicySubject.Builder()
      .addAttributes(subjectAttribute)
      .build();

PolicyRole policyRoles = new PolicyRole.Builder()
      .roleId("crn:v1:bluemix:public:cloud-object-storage::::serviceRole:ObjectWriter")
      .build();

ResourceAttribute accountIdResourceAttribute = new ResourceAttribute.Builder()
      .name("accountId")
      .value("ACCOUNT_ID")
      .operator("stringEquals")
      .build();

ResourceAttribute serviceNameResourceAttribute = new ResourceAttribute.Builder()
      .name("serviceName")
      .value("cloud-object-storage")
      .operator("stringEquals")
      .build();

PolicyResource policyResources = new PolicyResource.Builder()
      .addAttributes(accountIdResourceAttribute)
      .addAttributes(serviceNameResourceAttribute)
      .build();

CreatePolicyOptions options = new CreatePolicyOptions.Builder()
      .type("access")
      .subjects(Arrays.asList(policySubjects))
      .roles(Arrays.asList(policyRoles))
      .resources(Arrays.asList(policyResources))
      .build();

Response<Policy> response = service.createPolicy(options).execute();
Policy policy = response.getResult();

System.out.println(policy);
```
{: java}
{: codeblock}
   
```javascript
const policySubjects = [
  {
    attributes: [
      {
        name: 'iam_id',
        value: 'IBMid-123453user',
      },
    ],
  },
];
const policyRoles = [
  {
    role_id: 'crn:v1:bluemix:public:cloud-object-storage::::serviceRole:ObjectWriter',
  },
];
const accountIdResourceAttribute = {
  name: 'accountId',
  value: 'ACCOUNT_ID',
  operator: 'stringEquals',
};
const serviceNameResourceAttribute = {
  name: 'serviceName',
  value: 'cloud-object-storage',
  operator: 'stringEquals',
};
const policyResources = [
  {
    attributes: [accountIdResourceAttribute, serviceNameResourceAttribute]
  },
];
const params = {
  type: 'access',
  subjects: policySubjects,
  roles: policyRoles,
  resources: policyResources,
};

iamPolicyManagementService.createPolicy(params)
  .then(res => {
    examplePolicyId = res.result.id;
    console.log(JSON.stringify(res.result, null, 2));
  })
  .catch(err => {
    console.warn(err)
  });
```
{: javascript}
{: codeblock}
   
```python
policy_subjects = PolicySubject(
  attributes=[SubjectAttribute(name='iam_id', value='IBMid-123453user')])
policy_roles = PolicyRole(
  role_id='crn:v1:bluemix:public:cloud-object-storage::::serviceRole:ObjectWriter')
account_id_resource_attribute = ResourceAttribute(
  name='accountId', value='ACCOUNT_ID')
service_name_resource_attribute = ResourceAttribute(
  name='serviceName', value='cloud-object-storage')
policy_resources = PolicyResource(
  attributes=[account_id_resource_attribute,
        service_name_resource_attribute])

policy = iam_policy_management_service.create_policy(
  type='access',
  subjects=[policy_subjects],
  roles=[policy_roles],
  resources=[policy_resources]
).get_result()

print(json.dumps(policy, indent=2))
```
{: python}
{: codeblock}
   
```go
subjectAttribute := &iampolicymanagementv1.SubjectAttribute{
  Name:  core.StringPtr("iam_id"),
  Value: core.StringPtr("IBMid-123453user"),
}
policySubjects := &iampolicymanagementv1.PolicySubject{
  Attributes: []iampolicymanagementv1.SubjectAttribute{*subjectAttribute},
}
policyRoles := &iampolicymanagementv1.PolicyRole{
  RoleID: core.StringPtr("crn:v1:bluemix:public:cloud-object-storage::::serviceRole:ObjectWriter"),
}
accountIDResourceAttribute := &iampolicymanagementv1.ResourceAttribute{
  Name:     core.StringPtr("accountId"),
  Value:    core.StringPtr("ACCOUNT_ID"),
  Operator: core.StringPtr("stringEquals"),
}
serviceNameResourceAttribute := &iampolicymanagementv1.ResourceAttribute{
  Name:     core.StringPtr("serviceName"),
  Value:    core.StringPtr("cloud-object-storage"),
  Operator: core.StringPtr("stringEquals"),
}
policyResources := &iampolicymanagementv1.PolicyResource{
  Attributes: []iampolicymanagementv1.ResourceAttribute{
    *accountIDResourceAttribute, *serviceNameResourceAttribute}
}

options := iamPolicyManagementService.NewCreatePolicyOptions(
  "access",
  []iampolicymanagementv1.PolicySubject{*policySubjects},
  []iampolicymanagementv1.PolicyRole{*policyRoles},
  []iampolicymanagementv1.PolicyResource{*policyResources},
)

policy, response, err := iamPolicyManagementService.CreatePolicy(options)
if err != nil {
  panic(err)
}
b, _ := json.MarshalIndent(policy, "", "  ")
fmt.Println(string(b))
```
{: go}
{: codeblock}

## Assigning access to <service_name> by using Terraform
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
