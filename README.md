# Install IBM Storage Ceph
This repository includes serveral ways to install IBM Storage Ceph including connected and disconnected options. This is not a comprehensive archive for every possible installation option, it is intended only as a short list of common install options that can be adapated to many IBM Storage Ceph installations.

First let's cover the difference between a connected and a disconnected install. A connected install means you have direct internet access to the IBM Storage Ceph RPMs and containers. A disconnected install means that the installation must use mirrored repositories for both RPMs and containers (registry).

Note that Red Hat Enterprise Linux 8 or 9 is assumed to be installed, registered, and configured on all nodes prior to installing IBM Storage Ceph.

## Connected Install

### Simple (All OSDs are identical)
This install assumes that all nodes are essentially identical, and all storage is both identical and are flash. An all HDD install is not recommended. There are two configuration files provided for this install option. The first is ceph_cluster-spec-4-node-all-disks.yaml and is for an object only cluster with 4 nodes. 4 Nodes is the minimum supported number of nodes for IBM Storage Ceph.

1. Add IBM Storage Ceph RPM repo

**RHEL8**  
`# curl https://public.dhe.ibm.com/ibmdl/export/pub/storage/ceph/ibm-storage-ceph-5-rhel-8.repo | sudo tee /etc/yum.repos.d/ibm-storage-ceph-5-rhel-8.repo`

**RHEL9**  
`# curl https://public.dhe.ibm.com/ibmdl/export/pub/storage/ceph/ibm-storage-ceph-5-rhel-9.repo | sudo tee /etc/yum.repos.d/ibm-storage-ceph-5-rhel-9.repo`

2. Install and accept license

`# dnf -y install ibm-storage-ceph-license`  
`# touch /usr/share/ibm-storage-ceph-license/accept`

3. Ensure necessary packages are installed

`# dnf -y install cephadm ceph-common podman lvm2`


### Complex (OSDs are not identical or are compound)

## Disconnected Install (offline)
You will need to create a DNF/YUM repo to provide IBM Storage Ceph RPMs, and you will need to mirror the IBM Storage Ceph containers to a local registry.
