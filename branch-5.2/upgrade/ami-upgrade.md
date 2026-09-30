# Upgrade ScyllaDB Image: EC2 AMI, GCP, and Azure Images

To upgrade ScyllaDB images, you need to update:

1. ScyllaDB packages. Since ScyllaDB Open Source **5.2** and ScyllaDB
   Enterprise **2023.1**, the images are based on **Ubuntu 22.04**.
   See the [upgrade guide](https://opensource.docs.scylladb.com/branch-5.2/upgrade/index.md) for your ScyllaDB version
   for instructions for updating ScyllaDB packages on Ubuntu.
2. Underlying OS packages. ScyllaDB includes a list of 3rd party and OS packages
   tested with the ScyllaDB release.

To check your Scylla version, run the `scylla --version` command.
