---
title: Apache Cassandra backup and restore
linkTitle: Backup and restore tool
weight: 1
date: 2020-11-12T00:00:00.000Z
description: |
  Learn how to back up and restore Cassandra.
---

In an Apache Cassandra database cluster, data is replicated between multiple nodes and potentially between multiple datacenters. Cassandra is highly fault-tolerant, and depending on the size of the cluster, it can survive the failure of one or more nodes without any interruption in service. However, backups are still needed to recover from the following scenarios:

* Any errors made in data by client applications
* Accidental deletions
* Catastrophic failure that requires a rebuild of the entire cluster
* Rollback of the cluster to a known good state in the event of data corruption

This topic describes which Apache Cassandra data keyspaces to back up, and explains how to use scripts to create and restore the data in those keyspaces. It also describes which Apache Cassandra configuration and API Gateway configuration to back up.

{{% alert color="warning" title="Caution" %}}
You must read all of the following before you perform any of the instructions in this topic:

* These instructions apply only to an Apache Cassandra cluster where the replication factor is the same as the cluster size, which means that each node contains 100% of the data. This is the standard configuration for Axway API Management processes, and these instructions are intended to back up API Management keyspaces only.
* Because 100% of the data is stored on each node, you will run the backup procedure on a single node only, preferably on the seed node.
* The following instructions and scripts are intended as a starting point and can be customized and automated as needed to match your backup polices and environment.
* These instructions use the Cassandra snapshot functionality. A _snapshot_ is a set of hard links for the current data files in a keyspace.
  While the snapshot does not take up noticeable diskspace, it will cause stale data files not to be cleaned up. This is because a snapshot directory is created under each table directory and will contain hard links to all table data files. When Cassandra cleans up the stale table data files, the snapshot files will remain. Therefore, it is critical to remove them promptly. The example backup script takes care of this by deleting the snapshot files at the end of the backup.
* For safety reasons, the backup location should _not_ be on the same disk as the Cassandra data directory, and it also must have enough free space to contain the keyspace.


## Which data keyspaces to back up?

These procedures apply to data in API Management and KPS keyspaces only.

You must first obtain a list of the keyspace names to back up. API Management keyspaces may have a custom name defined, but they are named in the format of `<UUID>_group_[n]` by default. For example:

```
x9fa003e2_d975_4a4a_a27e_280ab7fd8a5_group_2p_2
```

All Cassandra internal keyspaces begin with `system`, and should not be backed up using this process. You can do this using `cqlsh` or `kpsadmin` commands:

### Find keyspaces using cqlsh

Using `cqlsh`, execute the following command:

```
SELECT * from system.schema_keyspaces;
```

In the following example, `xxx_group_2` and `xxx_group_3` are API Management keyspaces:

![API Management keyspaces](/Images/CassandraAdminGuide/cqlsh_keyspace.png)

### Find keyspaces using kpsadmin

Using `kpsadmin`, choose: `option 30) Show Configuration`, and enter the API Gateway group and any instance to back up, or use the command line, as shown in the following example:

![Find keyspaces using kpsadmin](/Images/CassandraAdminGuide/kpsadmin_keyspace.png)

## Download and configure the cassandra tool

### Download
TODO, waiting for Kelly input on where we are storing the tool.

### Configure
Once downloaded, you can add this tool on one node of your cluster, preferably on the seed node.

On the `conf` directory, you will find a file named `axway-cassandra-tool.ini` that you need to configure

|Element |Description|
|--------|:----------|
|cassandra_data_dir | Cassandra directory where your keyspaces are stored|
|cassandra_commitlog_dir | Cassandra directory where the commitlogs files are stored|
|cassandra_saved_caches_dir | Cassandra directory where the saved_caches files are stored|
|cqlsh_bin | Path to you Cqlsh bin|
|nodetool_bin | Path to you Nodetool bin|
|cqlsh_ip | The IP you use to connect through Cqlsh to you Cassandra|
|cql_username | Cqlsh username|
|cql_password | Cqlsh password|
|nodetool_username | Nodetool username|
|nodetool_password | Nodetool password|
|backup_root_dir | Path to your backup directory|
|printcmd | Set to true to see the command ran by the tool during the process|
|debug| Set to true to see debug logs|

Once you have set all your configuration, you can validate it by doing
```
axway-cassandra-tool validateConfig
```

This will validate if the tool can use you Cqlsh and Nodetool bin. You need a running Cassandra to make it work.

## Back up a keyspace

To back up a keyspace, you will use the backup command of the tool:

```
axway-cassandra-tool backup -k <your keyspace name> -s <your snapshot name>
```

You need to ensure that Cassandra is running for the processus to work.
Once done, you will get your backup with the following structure:

```
  <BACKUP_ROOT_DIR>
  ├── <BACKUP_SNAPSHOT_NAME>
  │ ├── <SNAPSHOT_NAME>
  │ │ ├── <TABLE_NAME>
  │ │ │  ├── <SNAPSHOT_FILES>
  │ │ ├── <TABLE_NAME>
  │ │ │  ├── <SNAPSHOT FILES>
  │ │ │...
```

Archive the `BACKUP_SNAPSHOT_NAME` directory using your company's archive method so you can restore it later if needed.


## Restore a keyspace

For the restore to be successful, the backup snapshot data must only contain data from the tables and columns in the keyspace schema. This is only necessary if the schema is not already present, for example, if setting up a new cluster as a copy of an existing cluster.

{{% alert title="Note" %}}
If you are restoring a keyspace to the same cluster that the backup was taken from, skip to [Steps to restore a keyspace](#steps-to-restore-a-keyspace).
{{% /alert %}}

Before you restore a keyspace in a new Cassandra cluster, you must ensure the following:

* The Cassandra cluster must be created to the API Gateway HA specifications. For more details, see [Configure a highly available Cassandra cluster](/docs/cass_admin/cassandra_config/).
* All API Gateway groups must have their schema created in the new cluster, and the replication factor must be the same as the cluster size (normally 3).
* If the keyspace name has changed in the new cluster, use the new name in the `KEYSPACE_NAME` variable in the restore script.
TODO, the new keyspace name is not an available option, so how do we do ?

### Steps to restore a keyspace

1. Shut down all API Gateway instances and any other clients in the Cassandra cluster.

2. Run the following command
```
axway-cassandra-tool restore -k <your keyspace name> -s <your snapshot name>
```

The tool will perform several task to restore your cluster, the whole process might take times, depending on the data you have on your backup.

Several time, the tool will ask you to perform manual actions on your nodes.

{{% alert title="Caution" %}}
Read carrefully the manual action, there are actions that need to be done on every node of your cluster and others that have to be executed only on the other nodes (not on the one where the script is running).
{{% /alert %}}

## Example: How to backup and restore a 3 node cluster into a new 3 node cluster

In order to make it work, you need to have a copy of **axway-cassandra-tool** into the seed node of the old cluster, and one copy into the new cluster.

1- Configure the **axway-cassandra-tool** for both cluster, check if the configuration is correct with
```
axway-cassandra-tool validateConfig
```

2- Run the backup script on your old Cluster
```
axway-cassandra-tool backup -k <your keyspace name> -s <your snapshot name>
```

3- Copy the backup folder in you new Cluster
The backup need to be copied into the **backup_root_dir** directory that you have configured in you new Cluster.

4- In your new Cluster, run the restore script

```
axway-cassandra-tool restore -k <your keyspace name> -s <your snapshot name>
```

The actions of starting and stopping Cassandra are not handled by the script, which mean you will need to perform them manually when the script ask you to.

## Example: How to restore on the same Cluster

In case your data become curropted, or you want to come back to a backup you've made in the past, on the same cluster, you only need to install your backup in the **backup_root_dir**  you configured, and run the restore command.

```
axway-cassandra-tool restore -k <your keyspace name> -s <your snapshot name>
```

If you still have the current keyspace in your Cassandra cluster, it will ask you if you want to remove it, and start fresh from the backup one.

{{% alert title="Caution" %}}
If you decide not to remove the keyspace, the script might fail and the data in your Cluster might become unstable.
{{% /alert %}}

## Which configuration to back up?

In addition to backing up your data in Apache Cassandra keyspaces, you must also back up your Apache Cassandra configuration and API Gateway configuration.

### Apache Cassandra configuration

You must back up the `CASSANDRA_HOME/conf` directory on all nodes.

### API Gateway group configuration

You must back up the API Gateway group configuration in the following directory:

```
API_GW_INSTALL_DIR/apigateway/groups/<group-name>/conf
```

This directory contains the API Gateway, API Manager, and KPS configuration data.
