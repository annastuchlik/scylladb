# Upgrade Guide - ScyllaDB Image 5.x.y to 5.x.z for EC2, GCP, and Azure

This document is a step-by-step procedure for upgrading from ScyllaDB Image 5.x.y to ScyllaDB Image 5.x.z, and rollback to 2021.1 if required.

## Applicable Versions

This guide covers upgrading ScyllaDB Image from version 5.x.y to version 5.x.z on EC2, GCP, and Azure. See [OS Support by Platform and Version](https://opensource.docs.scylladb.com/branch-5.1/getting-started/os-support.md) for information about supported versions.

## Upgrade Procedure

#### NOTE
Apply the following procedure **serially** on each node. Do not move to the next node before validating the node is up and running the new version.

A ScyllaDB upgrade is a rolling procedure which does **not** require full cluster shutdown.
For each of the nodes in the cluster, you will:

* Drain node and backup the data.
* Check your current release.
* Backup configuration file.
* Stop ScyllaDB.
* Download and install new ScyllaDB packages.
* Start ScyllaDB.
* Validate that the upgrade was successful.

**During** the rolling upgrade it is highly recommended:

* Not to use new 5.x.z features.
* Not to run administration functions, like repairs, refresh, rebuild or add or remove nodes.
* Not to apply schema changes.

## Upgrade steps

### Drain node and backup the data

Before any major procedure, like an upgrade, it is recommended to backup all the data to an external device. In ScyllaDB, backup is done using the `nodetool snapshot` command. For **each** node in the cluster, run the following command:

```sh
nodetool drain
nodetool snapshot
```

Take note of the directory name that nodetool gives you, and copy all the directories having this name under `/var/lib/scylla` to a backup device.

When the upgrade is complete (all nodes), the snapshot should be removed by `nodetool clearsnapshot -t <snapshot>`, or you risk running out of space.

### Backup configuration file

```sh
sudo cp -a /etc/scylla/scylla.yaml /etc/scylla/scylla.yaml.backup-5.x.z
```

### Gracefully stop the node

```sh
sudo service scylla-server stop
```

### Download and install the new release

Before upgrading, check what version you are running now using `dpkg -s scylla-server`. You should use the same version in case you want to [rollback](./#rollback-procedure) the upgrade. If you are not running a 5.x.y version, stop right here! This guide only covers 5.x.y to 5.x.z upgrades.

There are two alternative upgrade procedures:

* [Upgrading ScyllaDB and simultaneously updating 3rd party and OS packages](#upgrade-image-recommended-procedure). It is recommended if you are running a ScyllaDB official image (EC2 AMI, GCP, and Azure images), which is based on Ubuntu 20.04.
* [Upgrading ScyllaDB without updating any external packages](#upgrade-image-upgrade-guide-regular-procedure).

<a id="upgrade-image-recommended-procedure"></a>

**To upgrade ScyllaDB and update 3rd party and OS packages (RECOMMENDED):**

#### Versionadded
Added in version 5.0.

Choosing this upgrade procedure allows you to upgrade your ScyllaDB version and update the 3rd party and OS packages using one command.

1. Update the [ScyllaDB deb repo](https://www.scylladb.com/download/?platform=ubuntu-20.04&version=scylla-5.0) to 5.x.z.
2. Load the new repo:
   > ```sh
   > sudo apt-get update
   > ```
3. Run the following command to update the manifest file:
   > ```sh
   > cat scylla-packages-<version>-<arch>.txt | sudo xargs -n1 apt-get install -y
   > ```

   > Where:
   > > * `<version>` - The ScyllaDB version to which you are upgrading ( 5.x.z ).
   > > * `<arch>` - Architecture type: `x86_64` or `aarch64`.

   > The file is included in the ScyllaDB packages downloaded in the previous step. The file location is `http://downloads.scylladb.com/downloads/scylla/aws/manifest/scylla-packages-<version>-<arch>.txt`

   > Example:
   > > ```sh
   > > cat scylla-packages-5.1.2-x86_64.txt | sudo xargs -n1 apt-get install -y
   > > ```

   > > #### NOTE
   > > Alternatively, you can update the manifest file with the following command:

   > > `sudo apt-get install $(awk '{print $1'} scylla-packages-<version>-<arch>.txt) -y`

<a id="upgrade-image-upgrade-guide-regular-procedure"></a>

**To upgrade ScyllaDB:**

1. Update the [ScyllaDB deb repo](http://www.scylladb.com/download/) to 5.x.z.
2. Install:
   > ```sh
   > sudo apt-get update
   > sudo apt-get dist-upgrade scylla
   > ```

   > Answer ‘y’ to the first two questions.

### Start the node

```sh
sudo service scylla-server start
```

### Validate

1. Check cluster status with `nodetool status` and make sure **all** nodes, including the one you just upgraded, are in UN status.
2. Use `curl -X GET "http://localhost:10000/storage_service/scylla_release_version"` to check the ScyllaDB version.
3. Check the scylla-server log (execute `journalctl _COMM=scylla`) and `/var/log/syslog` to validate there are no errors.
4. Check again after 2 minutes, to validate no new issues are introduced.

Once you are sure the node upgrade is successful, move to the next node in the cluster.

## Rollback Procedure

#### NOTE
Execute the following commands one node at the time, moving to the next node only **after** the rollback procedure completed successfully.

The following procedure describes a rollback from ScyllaDB release 5.x.z to 5.x.y. Apply this procedure if an upgrade from 5.x.y to 5.x.z failed before completing on all nodes. Use this procedure only for nodes you upgraded to 5.x.z.

ScyllaDB rollback is a rolling procedure which does **not** require full cluster shutdown.
For each of the nodes rollback to 5.x.y, you will:

* Drain the node and stop ScyllaDB.
* Downgrade to previous release.
* Restore the configuration file.
* Restart ScyllaDB.
* Validate the rollback success.

Apply the following procedure **serially** on each node. Do not move to the next node before validating the node is up and running with the new version.

## Rollback steps

### Gracefully shutdown ScyllaDB

```sh
nodetool drain
sudo service scylla-server stop
```

### Downgrade to previous release

Install:

```sh
sudo apt-get install scylla=5.x.y\* scylla-server=5.x.y\* scylla-jmx=5.x.y\* scylla-tools=5.x.y\* scylla-tools-core=5.x.y\* scylla-kernel-conf=5.x.y\* scylla-conf=5.x.y\*
```

Answer ‘y’ to the first two questions.

### Restore the configuration file

```sh
sudo rm -rf /etc/scylla/scylla.yaml
sudo cp -a /etc/scylla/scylla.yaml.backup-5.x.z /etc/scylla/scylla.yaml
```

### Start the node

```sh
sudo service scylla-server start
```

### Validate

Check upgrade instruction above for validation. Once you are sure the node rollback is successful, move to the next node in the cluster.
