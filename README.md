# Install IBM Storage Ceph
This repository includes serveral ways to install IBM Storage Ceph including connected and disconnected options. This is not a comprehensive archive for every possible installation option, it is intended only as a short list of common install options that can be adapated to many IBM Storage Ceph installations.

First let's cover the difference between a connected and a disconnected install. A connected install means you have direct internet access to the IBM Storage Ceph RPMs and containers. A disconnected install means that the installation must use mirrored repositories for both RPMs and containers (registry).

Note that Red Hat Enterprise Linux 8 or 9 is assumed to be installed, registered, and configured on all nodes prior to installing IBM Storage Ceph.

## Connected Install

### Simple (All OSDs are identical)
This install assumes that all nodes are essentially identical, and all storage is both identical and are flash. An all HDD install is not recommended. There are two configuration files provided for this install option. The first is ceph_cluster-spec-4-node-all-disks.yaml and is for an *object only* cluster with 4 nodes. 4 Nodes is the minimum supported number of nodes for IBM Storage Ceph.

#### On All Nodes
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

#### On Admin Node
The `boostrap` command I will show assumes that the cluster has two networks, a public network, and a cluster network. If you only have one network just leave out the `--cluster-network` option. The IP addresses and ranges I am using are examples that you will need to replace with the IP addresses and ranges for your cluster. I am not showing how to build the `registry-auth.json` file. You will need to follow the IBM Storage Ceph documentation in order to create the file.

`# cephadm bootstrap --mon-ip 192.168.100.201 --cluster-network 192.168.140.0/24 --registry-json /root/registry-auth.json --apply-spec /root/ceph_cluster-spec.yaml`

### Complex (OSDs are not identical or are compound)

## Disconnected Install (offline)
You will need to create a DNF/YUM repo to provide IBM Storage Ceph RPMs, and you will need to mirror the IBM Storage Ceph containers to a local registry.
