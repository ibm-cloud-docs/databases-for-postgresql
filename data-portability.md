---

copyright:
  years: 2024
lastupdated: "2024-11-11"

keywords: export data, portability, pgdump, pgadmin

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}



# Understanding data portability for {{site.data.keyword.databases-for-postgresql}}
{: #data-portability}

[Data portability](#x2113280){: term} involves a set of tools, and procedures that enable customers to export the digital artifacts that would be needed to implement similar workload and data processing on different service providers or on-prem software. It includes procedures for copying and storing the service customer's content, including the related configuration used by the service to store and process the data, on customer's own location.
{: shortdesc}

## Responsibilities
{: #data-portability-responsibilities}

{{site.data.keyword.cloud}} services provide interfaces and instructions to guide the customer to copy and store the service customer content, including the related configuration, on their own selected location.

The customer then is responsible for the use of the exported data and configuration for the purpose of data portability to other infrastructures.
This can involve:

- the planning and execution for setting up alternate infrastructure on on different cloud providers or on-prem software that provide similar capabilities to the IBM services
- the planning and execution for the porting of the required application code on the alternate infrastructure, including the adaptation of customer's application code, deployment automation, etc.
- the conversion of the exported data and configuration to format required by the alternate infrastructure and adapted applications.


To find out more about responsibility ownership for using {{site.data.keyword.cloud_notm}} products between {{site.data.keyword.IBM_notm}} and customer see [Shared responsibilities for {{site.data.keyword.cloud_notm}} products](/docs/overview?topic=overview-shared-responsibilities).



## Data export procedures
{: #data-portability-procedures}

{{site.data.keyword.databases-for-postgresql}} provides mechanisms to export your content that has been uploaded, stored, and processed using the service.

## Migrating data from {{site.data.keyword.databases-for-postgresql}}
{: #data-portability-migrating-data-postgresql}

You can use the following methods to export data from {{site.data.keyword.databases-for-postgresql}}.

Connect to your {{site.data.keyword.cloud}} deployment: 

To access your {{site.data.keyword.databases-for-postgresql}} deployment and its tools, follow the connection instructions provided in our documentation. Once connected, you'll have access to the `psql` and `pg_dump` commands. You can also use PGadmin to export data. For more information, see the [Getting started page](/docs/databases-for-postgresql?topic=databases-for-postgresql-getting-started&interface=ui).

Ensure that you are connected to the deployment containing the database you want to export. Replace `<<CRN>>` with your actual Cloud Resource Name.

```sh
ibmcloud cdb cxn <<CRN>> -s
```
{: pre}

## Using `pg_dump`
{: #ata-portability-pg_dump}

On your database run `pg_dump` to create an SQL file, which can be used to re-create the database. At a minimum, `pg_dump` takes a hostname (`-h` flag), port number (`-p` flag), database name (`-d` flag), username (`-U` flag), and a file (or directory name) to write the dump to (`-f` flag). 

For example, the following command dumps the {{site.data.keyword.databases-for-postgresql}} "compose" database that is hosted on sl-eu-lon-2-portal.4.dblayer.com, port 17980, using the admin user and save the results in `dump.sql`.

```sh
pg_dump -h sl-eu-lon-2-portal.4.dblayer.com -p 17980 -d compose -U admin -f dump.sql
```
{: pre}

Further options:
The `pg_dump` command offers more capabilities. Please refer to the [official documentation](https://www.postgresql.org/docs/current/backup-dump.html) and [command reference](https://www.postgresql.org/docs/current/app-pgdump.html) for a complete list and detailed explanation. You can export specific parts of your database instead of the entire structure using the documented options.
  
Dumps can be generated as either script or archive files (use `t` option). Script dumps are plain-text SQL commands intended to be read with `psql`, while archive file dumps require `pg_restore` for reconstruction. Archive formats offer greater flexibility, allowing for selective restoration.
{: .note}

 ### Additional migration option by using pg_restore
{: #pg_restore}
  
 For TAR files containing separate SQL and data files, the `pg_restore` command provides a more flexible approach to database migration.
 An example of the `pg_restore` command is:

```sh
 PGPASSWORD=yourpasswordhere PGSSLROOTCERT=cert.crt pg_restore -h c7798cf6-e5d2-4513-b17f-3d3fa67d8291.8f7bfd8f3faa4218aec56e069eb46187.databases.appdomain.cloud -p 32484 -U admin -F t -d ibmclouddb tarfile.tar
```
{: .pre}

## Exported data formats
{: #data-portability-data-formats}

The exported data can be in plain text (`sql`) or archive files (`tar`), and based on the file formats, data can be migrated to any other Postgresql instance using either `psql` or `pg_restore` commands. To restore data from a TAR file, see the [pg_restore](https://www.postgresql.org/docs/current/app-pgrestore.html) documentation.
  
## Data ownership
{: #data-ownership}

All exported data are classified as Customer content and therefore apply to them the full customer ownership and licensing rights, as stated in [{{site.data.keyword.cloud}} Service Agreement](https://www.ibm.com/terms/?id=Z126-6304_WS).
