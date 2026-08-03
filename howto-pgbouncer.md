---

copyright:
  years: 2026
lastupdated: "2026-08-03"

keywords: postgresql, databases, pgbouncer, connection pooling, connection pooler, auth_query, auth_user, pgbouncer_auth, scram-sha-256, credential rotation, transaction pooling

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Connection pooling with PgBouncer
{: #pgbouncer}

[PgBouncer](https://www.pgbouncer.org/){: external} is a lightweight connection pooler for PostgreSQL. It maintains a small number of database connections and shares them across many application clients, which helps your deployment within its [connection limit](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections#connection-pooling) and avoids the overhead of opening a new connection for every client.

{{site.data.keyword.databases-for-postgresql_full}} deployments include built-in support for PgBouncer's [`auth_query`](https://www.pgbouncer.org/config.html){: external} authentication. Every deployment provides a `public.pgbouncer_lookup` function and a `pgbouncer_auth` role, so a PgBouncer instance that you run can validate database users directly against your deployment. You do not maintain a local password list for your database users, and password changes take effect immediately, with no PgBouncer restart or reload.

{{site.data.keyword.databases-for-postgresql}} does not host or operate PgBouncer. You install, run, secure, and update PgBouncer on your own infrastructure, such as a virtual server, a Kubernetes cluster, or an application sidecar. For more information about connection management, see [Managing connections](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections&interface=ui).
{: important}

## Before you begin
{: #pgbouncer-before-begin}

You need:

- A {{site.data.keyword.databases-for-postgresql}} deployment with the [admin password set](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#user-admin).
- A database user for your application, [created in the UI, CLI, or API](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#creating_users).
- PgBouncer 1.11.0 or later, which supports SCRAM authentication, installed on infrastructure that you control. If you plan to use protocol-level prepared statements in transaction pooling mode, use PgBouncer 1.21.0 or later.
- A [`psql` client](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql).
- Your deployment connection information:
  - Hostname and port, from the [connection strings]...
  - CA certificate, retrieved with `ibmcloud cdb deployment-cacert`.

Support for `pgbouncer_lookup` is rolling out across deployments. To confirm that your deployment has the function, connect with `psql` as `admin` and run `\df public.pgbouncer_lookup`. If the result is empty, your deployment receives the function with an upcoming maintenance update.
{: note}

## Creating a dedicated authentication user
{: #pgbouncer-auth-user}

PgBouncer runs `auth_query` as one designated login role, its `auth_user`. Create a role that is used only for this purpose. A dedicated role that owns no data and runs nothing else is the least-privilege choice.

Connect with `psql` as the `admin` user, then create the role and grant it `pgbouncer_auth`:

```sql
CREATE ROLE pool_auth WITH LOGIN PASSWORD '<POOL_AUTH_PASSWORD>';
GRANT pgbouncer_auth TO pool_auth;
```
{: pre}

The `pgbouncer_auth` role carries a single privilege: permission to run the `pgbouncer_lookup` function. The `admin` user holds `pgbouncer_auth` with the admin option, so you grant and revoke membership yourself. You can also create the user with `ibmcloud cdb user-create` if you want it to appear in your service credentials, but [users created that way](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#user-management-cli) are members of `ibm-cloud-base-user` and can create users and databases, which is more than the auth user needs.

Grant `pgbouncer_auth` only to the dedicated auth user. Any member of the role can read the stored password verifiers of your other database users, so every additional member widens the impact of a compromised credential.
{: important}

To retire an auth user, revoke its membership:

```sql
REVOKE pgbouncer_auth FROM pool_auth;
```
{: pre}

## Configuring PgBouncer
{: #pgbouncer-configure}

The following PgBouncer settings are required when connecting to a {{site.data.keyword.databases-for-postgresql}} deployment:

- `auth_type = scram-sha-256`, because deployments store SCRAM-SHA-256 password verifiers.
- `auth_query = SELECT * FROM public.pgbouncer_lookup($1)`, because PgBouncer's default `auth_query` reads `pg_authid` directly and your database users cannot read it.
- Server-side TLS, because deployments only accept TLS connections. Set `server_tls_sslmode = verify-full`, and save the certificate that you retrieved with `ibmcloud cdb deployment-cacert` to the path that you set in `server_tls_ca_file`.

PgBouncer reads the credentials of its own `auth_user` from `auth_file`, so in this configuration, userlist.txt contains only the auth_user credentials. Every other user resolves through `auth_query`. Restrict the file's permissions to the PgBouncer process owner, for example with mode `0600`.

```txt
"pool_auth" "<POOL_AUTH_PASSWORD>"
```
{: codeblock}

A complete minimal configuration, where `<HOSTNAME>` and `<PORT>` come from your [connection strings](/docs/databases-for-postgresql?topic=databases-for-postgresql-connection-strings):

```ini
[databases]
ibmclouddb = host=<HOSTNAME> port=<PORT> dbname=ibmclouddb

[pgbouncer]
listen_addr = 127.0.0.1
listen_port = 6432

auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt
auth_user = pool_auth
auth_query = SELECT * FROM public.pgbouncer_lookup($1)

server_tls_sslmode = verify-full
server_tls_ca_file = /etc/pgbouncer/ca-certificate.crt

pool_mode = session
max_client_conn = 200
default_pool_size = 20
```
{: codeblock}

Do not set `auth_dbname = postgres`. The lookup function is not installed in the `postgres` database. Leave `auth_dbname` unset, so the authentication query runs in the database that the client connects to. Databases that you create later include the function automatically.
{: note}

For descriptions of every setting, see the [PgBouncer configuration reference](https://www.pgbouncer.org/config.html){: external}.

## Verifying the setup
{: #pgbouncer-verify}

1. Confirm that the lookup resolves one of your database users. Connect with `psql` as `admin` and run:

    ```sql
    SELECT usename, passwd IS NOT NULL AS can_authenticate
      FROM public.pgbouncer_lookup('<APP_USERNAME>');
    ```
    {: pre}

    The result is one row with `can_authenticate = t`. The query is written so that the password verifier is not displayed. Service-reserved users, including `admin`, return no rows.

1. Connect through PgBouncer as the database user:

    ```sh
    psql "host=127.0.0.1 port=6432 dbname=ibmclouddb user=<APP_USERNAME>"
    ```
    {: pre}

   If the connection succeeds, PgBouncer is correctly authenticating users through `auth_query`.

1. Rotate the database user's password, then reconnect through PgBouncer with the new password:

    ```sh
    ibmcloud cdb user-password <DEPLOYMENT_NAME_OR_CRN> <APP_USERNAME> <NEW_PASSWORD>
    ```
    {: pre}

    The new password works immediately. No PgBouncer restart, reload, or `userlist.txt` change is needed.

## How the security model works
{: #pgbouncer-security-model}

The pgbouncer_lookup function exposes only the information that PgBouncer requires for authentication.

- The function runs with [`SECURITY DEFINER`](https://www.postgresql.org/docs/current/sql-createfunction.html#SQL-CREATEFUNCTION-SECURITY){: external} and a pinned `search_path`, and reads `pg_catalog.pg_authid` on your behalf. Direct access to `pg_authid` and `pg_shadow` stays blocked.
- It returns hashed SCRAM verifiers, never plain-text passwords.
- Only roles that are allowed to log in resolve. Service-reserved users, such as `admin` and the internal replication and operations users, never resolve.
- If a role's [`VALID UNTIL`](https://www.postgresql.org/docs/current/sql-createrole.html){: external} timestamp is in the past, the function returns a NULL password, so authentication fails and password expiry stays enforced.
- Permission to run the function is revoked from `PUBLIC` and granted only to `admin` and to members of `pgbouncer_auth`.

PgBouncer's default `auth_query` reads `pg_authid` directly, which your database users cannot do. For this situation, the [PgBouncer documentation](https://www.pgbouncer.org/config.html){: external} recommends calling a `SECURITY DEFINER` function through a non-superuser instead. The `pgbouncer_lookup` function is that function, with the service's reserved users excluded.

## Limitations and considerations
{: #pgbouncer-limitations}

- PgBouncer does not raise your deployment's `max_connections`. Size `default_pool_size` so that the total number of server connections from all your PgBouncer instances stays within the [connection limit](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections#connection-pooling). If you reach the limit, see [increasing max connections](/docs/databases-for-postgresql?topic=databases-for-postgresql-troubleshoot-max-connect).
- The `admin` user cannot authenticate through `auth_query`. For administrative tasks, connect as `admin` directly to the deployment, not through PgBouncer.
- The `postgres` database does not have the lookup function, so never set `auth_dbname = postgres`.
- In transaction pooling mode (`pool_mode = transaction`), session state such as `SET` variables, temporary tables, advisory locks, and `LISTEN` channels does not carry over between transactions. Protocol-level prepared statements require `max_prepared_statements` and PgBouncer 1.21.0 or later. For more information, see [PgBouncer features](https://www.pgbouncer.org/features.html){: external}.
- During an [in-place major version upgrade](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading#upgrading-in-place), your deployment experiences a brief period of downtime and open connections are terminated (SQLSTATE `57P01`). PgBouncer re-establishes its server connections automatically, but your applications must retry interrupted transactions and re-prepare statements.
- Read-only users cannot connect to the primary endpoint. To pool their connections, add a separate `[databases]` entry that points at your [read-only replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas).

## Next steps
{: #pgbouncer-next-steps}

- [Managing connections](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections)
- [Getting connection strings](/docs/databases-for-postgresql?topic=databases-for-postgresql-connection-strings)
- [Managing users, roles, and privileges](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management)
- [PgBouncer configuration reference](https://www.pgbouncer.org/config.html){: external}
