---

Copyright:
  years: 2019
lastupdated: "2019-01-14"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}


# Using Logical Replication

{{site.data.keyword.databases-for-postgresql_full}} supports [logical replication](https://www.postgresql.org/docs/current/logical-replication.html) between your deployment and an external PostgreSQL instance. You can set up your external PostgreSQL as a publisher, your {{site.data.keyword.databases-for-postgresql}} deployment as a subscriber, and replicate your data across from an external database into your deployment.

Logical Replication is only available on deployments running PostgreSQL 10 or above.
{: .tip}

## Configuring the Publisher

The external PostgreSQL instance is the publisher, and needs to be configured in order for your {{site.data.keyword.databases-for-postgresql}} deployment to connect and be able to pull the data in correctly.

1. Every table that is selected for replication needs to contain a [Primary Key](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-PRIMARY-KEYS).
2. Your external (publisher) PostgreSQL needs to have an `admin` user, and that user needs to have privileges on the databases you would like replicate.
    ```bash
        postgres=> CREATE ROLE admin WITH REPLICATION LOGIN PASSWORD 'my_password';
        postgres=> GRANT ALL PRIVILEGES ON DATABASE example-database TO admin;
        postgres=> GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO admin;
    ```
3. The PostgreSQL you are replicating from needs to have TLS/SSL enabled.

### Enabling TLS/SSL

In order for PostgreSQL to start and run with TLS/SSL enabled, it must have files containing a server certificate and private key.

- Navigate to your local PostgreSQL data directory.
- Create self-signed certificate with `openssl` by running the following command. Make sure to set `Common Name` to your local PostgreSQL hostname or public IP address.
    ```bash
    openssl req -new -text -out server.crt -keyout server.key -subj "/CN=dbhost.yourdomain.com"
    ```
- You now have two files that have been generated, `server.crt` and `server.key`
- Remove the pass phrase from the generated private key
    ```bash
    openssl rsa -in server.key -out server.key
    ```
- Modify the permissions on the created key
    ```bash
    chmod og-rwx server.key
    ``` 
    If the permissions are more liberal, the server rejects the file.
- Edit your local `postgresql.conf` and set `ssl=on`
- Restart the PostgreSQL server.
- Verify TLS/SSL by connecting with `psql`. You should see a similar message upon connecting.
    ```
    psql (10.4, server 10.5)
    SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
    Type "help" for help.
    ```

## Configuring the Subscriber

To configure your {{site.data.keyword.databases-for-postgresql}} deployment and to make sure that your data is replicated across correctly,

1. You need to create a database in your deployment with same name as the database you intend to replicate.
2. Logical Replication works at the table level, so every table you select to publish, you need to create in the subscriber manually before starting the logical replication process. The table on the subscriber does not need to be identical to its publisher counterpart. However, the table on the subscriber must contain at least every  column present in the table on the publisher. Additional columns present in the subscriber must not have NOT NULL or other constraints. If they do, replication fails.

**Note** Native PostgreSQL subscription commands require superuser privileges, which are not available on {{site.data.keyword.databases-for-postgresql}} deployments. Instead, your deployment includes a set of functions that can be used to set and manage logical replication for the subscription. 

Only the [admin user](./admin-connecting.html) that is provided by {{site.data.keyword.databases-for-postgresql}} has permissions to run the following replication commands that allow you to subscribe and replicate content from an external PostgreSQL publisher.
{: .tip}

### Subscriber Functions

**`create_subscription`**
```bash
Arguments:
    subscription_name   Unique name to create the subscription channel with
    host_ip             Publisher hostname or public IP address
    port                Port number publisher is running on
    username            `admin` user created on the publisher
    password            Password of the `admin` user on the publisher
    db_name             The name of the database to be replicated
    publisher_name      The name of publisher channel on the publisher

Usage:
    example-database=> SELECT create_subscription('subs1','130.215.223.184','5432','password','admin','example-database','my_publication');
```

**`delete_subscription`**
```bash
Arguments:
    subscription_name   Name the subscription channel to delete
    db_name             The name of the replicated database

Usage:
example-database=> SELECT delete_subscription('subs1', 'example-database');
```

**`list_subscriptions`**
```bash
Arguments:
    None

Usage:
    example-database=> SELECT * FROM list_subscriptions();
```

**`disable_subscription`**
```bash
Arguments:
    subscription_name   Name the subscription channel to disable
    db_name             The name of the replicated database

Usage:
    example-database=> SELECT disable_subscription('subs1','example-database');
```

**`enable_subscription`**
```bash
Arguments:
    subscription_name   Name the subscription channel to enable
    db_name             The name of the replicated database

Usage:
    example-database=> SELECT enable_subscription('subs1','example-database');
```

**`refresh_subscription`**  

This function is used to refresh a subscription on the subscriber after changes are made on the publisher, like adding or removing a table.
```bash
Arguments:
    subscription_name   Name the subscription channel to refresh
    db_name             The name of the replicated database

Usage:
    example-database=> SELECT refresh_subscription('subs1','example-database');
```

## Setting up Logical Replication

### On the Publisher

To configure your external PostgreSQL as a publisher,

- Edit your local `pg_hba.conf` and add the following 
    ```text
    hostssl    replication            all         0.0.0.0/0      md5
    hostssl    all                    all         0.0.0.0/0      md5
    ```

- Edit your local `postgresql.conf` with the required [logical replication configuration](https://www.postgresql.org/docs/10/logical-replication-config.html). Set `wal_level` to 'logical', and set `listen_addresses='*'` to accept connections from any host.  
    ```text
    listen_addresses='*'
    wal_level = logical                   
    ```

- Restart your PostgreSQL server.

Now you can define a publisher on the database and add the tables you want to replicate to the subscriber.

- Log in to database you want to publish from with the `admin` user
    ```bash
        psql -U admin -d example-database
    ```
- Create the publication channel
    ```bash
        example-database=> CREATE PUBLICATION my_publication;
    ``` 
- Add tables to publisher
    ```bash
       example-database=> ALTER PUBLICATION my_publication ADD TABLE my_table;
    ```

### On the Subscriber

To configure your {{site.data.keyword.databases-for-postgresql}} as a subscriber,
- Log in to the database created for replication with `admin` user.
    ```bash
    psql -U admin -d example-database
    ```
- Run the following query to call the `create_subscription` function and create the subscriber channel. 
    ```bash
        example-database=> SELECT create_subscription('subs1','130.215.223.184','5432','admin','password','example-database','my_publication');
    ```

## Monitoring Replication

You can monitor the status of logical replication from both the publisher and the subscriber by running the following query on either.
```bash
    example-database=> SELECT * FROM pg_stat_replication;
```
