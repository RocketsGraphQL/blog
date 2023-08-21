---
title: "Install pgAudit in your AWS RDS instance"
description: "Install and enable pgAudit in AWS RDS instance for logging your Postgres events"
pubDate: "2023-08-21"
hero: "/images/rocketgraph-logs.gif"
tags: ["astro"]
layout: "../../layouts/BlogPostLayout.astro"
---

## Basic setup

First we'll have to create an AWS RDS DB for this. We'll use minimal permissions for this setup so that we can easily understand what's going on.

### Step 1: Create Database 

Create your AWS account. Go to AWS RDS tab and click on "Create Database"

### Step 2: Select Database

Standard create and postgreSQL

<img title="a title" alt="Alt text" src="../../../public/images/rds/db-1.png">

Select pg Version to the latest

<img title="a title" alt="Alt text" src="../../../public/images/rds/pg-version.png">

And use free tier for now

### Step 3: Add credentials

<img title="a title" alt="Alt text" src="../../../public/images/rds/cred.png">

Username: postgres
MasterPassword: postgres

For now.

### Step 4: Allocate storage

Allocate the minimum possible storage for now

<img title="a title" alt="Alt text" src="../../../public/images/rds/storage.png">

### Step 5: Network

<img title="a title" alt="Alt text" src="../../../public/images/rds/vpc-1.png">

Create a new VPC for this RDS DB and create new security groups. Make ssure that they allow all traffic for now. Let's improve the security later

### Step 6: Set public access

<img title="a title" alt="Alt text" src="../../../public/images/rds/connect.png">

Remember to set Public access to true since we want to login from psql.

### Step 7: DB Parameter group

<img title="a title" alt="Alt text" src="../../../public/images/rds/ad-config-1.png">

Remember the DB Parameter group name since we need to tweak it later for installing pgAudit

### Step 8: Configure logging

This is the important part

<img title="a title" alt="Alt text" src="../../../public/images/rds/log-exports.png">

Configure log exports to Cloudwatch. Create a service linked role with necessary permissions if you haven't. Give it admin access for now.


That's it now click on "create database" and wait for the instance to be available

### Step 9: Installing pgAudit

Follow this video for setting up your DB Parameter group, installing pgAudit and enabling it.

[![Everything Is AWESOME](https://img.youtube.com/vi/in-k0Jf2SnU/0.jpg)](https://www.youtube.com/watch?v=in-k0Jf2SnU "Everything Is AWESOME")

To load pgAudit, you need to configure the DB Parameter group to the following:

Set pgaudit.loog to `none`, we'll change this later from psql:
<img title="a title" alt="Alt text" src="../../../public/images/rds/log.png">

Load shared libraries via `shared_preload_libraries`:

<img title="a title" alt="Alt text" src="../../../public/images/rds/shared.png">

Set the pgaudot.role to the `rds_pgaudit`

<img title="a title" alt="Alt text" src="../../../public/images/rds/role.png">



Restart the instance and you should be able to use pgAudit


Log into your RDS using psql using

```bash
psql postgresql://postgres:password@small-sound.czei94v3hlkf.us-east-2.rds.amazonaws.com:5432/postgres

```

Then create a role for pgAudit

```sql
postgres=> CREATE ROLE rds_pgaudit; 
```

Check that the libraries are loaded

```bash
postgres=> show shared_preload_libraries;
```

Then enable the extension

```bash
postgres=> CREATE EXTENSION pgaudit;
```

Enable logs for pgAudit

For now, for testing purposes set log level to CREATE

```bash
postgres=> ALTER DATABASE test_database set pgaudit.log="CREATE"; 
```

But ideally, it should be

```bash
postgres=> ALTER DATABASE test_database set pgaudit.log="ALL"; 
```

### Step 10: Testing that pgAudit works

In the psql shell

```bash
postgres=> CREATE TABLE test_table (id int);
postgres=> SELECT * FROM test_table;
```

Go to logs & events tab 

<img title="a title" alt="Alt text" src="../../../public/images/rds/logs-tab.png">

and check that the logs are reflected by clicking view on the latest written log file

<img title="a title" alt="Alt text" src="../../../public/images/rds/logs-selection.png">


## Rocketgraph setup

Alternatively you can create a project using Rocketgraph that comes with pgAudit enabled. You just need to do step 9. And once you create extension, you can see all your logs nicely like this:

<img title="a title" alt="Alt text" src="../../../public/images/rocketgraph-logs.gif">
