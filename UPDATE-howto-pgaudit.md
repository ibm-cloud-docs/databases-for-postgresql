---

copyright:
  years: 2020
lastupdated: "2020-07-30"

keywords: postgresql, databases, pgaudit, logging, session, object, pg role

subcollection: databases-for-postgresql

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# pgAudit
{: #pgaudit}

{{site.data.keyword.databases-for-postgresql_full}} allows enablement of both session logging and object logging.  There are some considerations to take into account.

### Session Logging

Straightforward.  This allows us to set a set of audit events that get logged for all activity.  Currently I've implemented as a setting for the whole DB cluster via a security definer function that the `admin` user can call.  We could also allow it to be set on a per user basis or per databases basis.  Per user might be a useful addition, per-database probably less so.

### Object Logging

This is far more nuanced and requires quite a bit of knowledge on how the PG role system works to set this properly.  That means we are going to need good usage docs ( 👋  @jodonn )

Object logging works by created a PG Role, telling pgAudit to use that role for auditing, and then granting permissions on DB objects to on that audit role.  Pretty straightforward, but here is where it gets fun.

A user owns their own DB objects.  So that user can grant and revoke permissions from their own objects.  This is probably best documented with an example.

We have this shift person named `brad` that we want to audit some DB activity on with object logging.

`brad` owns the table secret.  As table owner, `brad` has full access to this table.
```
ibmclouddb=> \dt
        List of relations
 Schema |  Name  | Type  | Owner
--------+--------+-------+-------
 public | secret | table | brad
(1 row)
```  

We want to audit every time `brad` looks at the `secret.pw` column.

So we login as `admin`, and enable object logging on that column:

```
ibmclouddb=> select enable_pgaudit_object_logging();
 enable_pgaudit_object_logging
-------------------------------
 ok
(1 row)

ibmclouddb=> GRANT SELECT(pw) ON public.secret to pgaudit ;
GRANT
```

Now log back in as `brad` and snoop around:

```
ibmclouddb=> select * from secret ;
 uname |     pw
-------+-------------
 brad  | supersecret
(1 row)
```

And in the DB logs, we see what we want:
`2020-07-30 20:21:19 UTC [psql] [00000] [3518]: [1-1] user=brad,db=ibmclouddb,client=127.0.0.1 LOG:  AUDIT: OBJECT,1,1,READ,SELECT,TABLE,public.secret,select * from secret ;,<not logged>`

However, because `brad` owns the table and has full access, `brad` can disable audit logging as follows.  Still logged in as `brad`:

```
ibmclouddb=> REVOKE SELECT ON secret FROM pgaudit ;
REVOKE
ibmclouddb=> select * from secret ;
 uname |     pw
-------+-------------
 brad  | supersecret
(1 row)
```

And there is no audit log of the event.  

Obviously, allowing users to disable their own audit logging limits the usefulness of auditing to approximately nothing.  But there is a solution.  But that solution is a complex one, that involves changing ownership and permissions of the tables.  Here is how to lock down the `secret` table.

Log back in as `admin`.

`admin` is able to grant itself access to other roles. So we have to grant access to `brad` first:

```
ibmclouddb=> grant brad TO admin ;
GRANT ROLE
```

Then we need to change the ownership of the table `secret` from `brad` to `admin`.

```
ibmclouddb=> ALTER TABLE secret OWNER TO admin ;
ALTER TABLE
ibmclouddb=> \dt
        List of relations
 Schema |  Name  | Type  | Owner
--------+--------+-------+-------
 public | secret | table | admin
(1 row)
```

At this point, we have locked `brad` out of the table.  `brad` relied on table ownership to be able to access the table, and we removed that.  So we need to grant access back to `brad`.  We will grant all privileges back - as that is what access previously was.  We will also enable auditing again:

```
ibmclouddb=> grant ALL on secret TO brad;
GRANT
 GRANT SELECT(pw) ON public.secret to pgaudit ;
GRANT
```

Log back in as `brad` and try to break things again:
```
ibmclouddb=> select * from secret ;
 uname |     pw
-------+-------------
 brad  | supersecret

(this was audit logged).

ibmclouddb=> revoke ALL ON secret FROM pgaudit ;
WARNING:  no privileges could be revoked for "secret"
WARNING:  no privileges could be revoked for column "tableoid" of relation "secret"
WARNING:  no privileges could be revoked for column "cmax" of relation "secret"
WARNING:  no privileges could be revoked for column "xmax" of relation "secret"
WARNING:  no privileges could be revoked for column "cmin" of relation "secret"
WARNING:  no privileges could be revoked for column "xmin" of relation "secret"
WARNING:  no privileges could be revoked for column "ctid" of relation "secret"
WARNING:  no privileges could be revoked for column "uname" of relation "secret"
WARNING:  no privileges could be revoked for column "pw" of relation "secret"
REVOKE
```

This is complex, non-obvious and requires a good understanding the the PG role system (which is complex and non-obvious itself).  

I don't expect our customers to understand this.  We will need to document it heavily, and I suspect ensure that @Andrew-Rombach fully understands how this works and is prepared to work with some of our larger customers that are going to want to implement this.