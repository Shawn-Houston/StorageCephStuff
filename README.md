# Install IBM Storage Ceph
This repository includes serveral ways to install IBM Storage Ceph including connected and disconnected options. This is not a comprehensive archive for every possible installation option, it is intended only as a short list of common install options that can be adapated to many IBM Storage Ceph installations.

First let's cover the difference between a connected and a disconnected install. A connected install means you have direct internet access to the IBM Storage Ceph RPMs and containers. A disconnected install means that the installation must use mirrored repositories for both RPMs and containers (registry).

Note that Red Hat Enterprise Linux 8 or 9 is assumed to be installed, registered, and configured on all nodes prior to installing IBM Storage Ceph.

## Connected Install

### For All Install Types

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

### Simple (All OSDs are identical)
This install assumes that all nodes are essentially identical, and all storage is both identical and are flash. An all HDD install is not recommended. There are two configuration files provided for this install option. The first is ceph_cluster-spec-4-node-all-disks.yaml and is for an *object only* cluster with 4 nodes. 4 Nodes is the minimum supported number of nodes for IBM Storage Ceph.

#### On Admin Node
The `boostrap` command I will show assumes that the cluster has two networks, a public network, and a cluster network. If you only have one network just leave out the `--cluster-network` option. The IP addresses and ranges I am using are examples that you will need to replace with the IP addresses and ranges for your cluster. I am not showing how to build the `registry-auth.json` file. You will need to follow the IBM Storage Ceph documentation in order to create the file.

`# cephadm bootstrap --mon-ip 192.168.100.201 --cluster-network 192.168.140.0/24 --registry-json /root/registry-auth.json --apply-spec /root/ceph_cluster-spec-4-node-all-disks.yaml`

When boostrap completes it will print out the Console URL, the admin username, and the temporary admin password. The installation will continue for some time. Use this time to login to the Console and update the admin password. You can then watch the installation proceed in the console, or you can use the CLI:

`# ceph orch ps`

There will be 4 RGWs once the cluster finishes installing. You will want to install a proxy and configure it to load balance object requests across the 4 RGW endpoints. That is left as an excersice for the reader.

### Complex (OSDs are not identical or are compound)
For this install we will be targeting a mixed HDD and flash cluster and we will be configuring 3 protocols, RBD, cephFS, and object (RGWs). The example has a number of HDDs per node, not to exceed the fan out of the single flash (SSD) that will provide the WAL and Rocks DB for the HDDs. This is typical of an "active archive" cluster, or a medium performance cluster providing External Mode to Fusion Data Foundation as part of an IBM Storage Fusion install. This cluster will have 5 nodes, which is what I consider to be the minimum number of nodes when providing all 3 of RBD, cephFS, and object. As for the previous 4 node install, we start with the bootstrap.

#### On Admin Node
The `boostrap` command I will show assumes that the cluster has two networks, a public network, and a cluster network. If you only have one network just leave out the `--cluster-network` option. The IP addresses and ranges I am using are examples that you will need to replace with the IP addresses and ranges for your cluster. I am not showing how to build the `registry-auth.json` file. You will need to follow the IBM Storage Ceph documentation in order to create the file.

`cephadm bootstrap --mon-ip 192.168.100.201 --cluster-network 192.168.140.0/24 --registry-json /root/registry-auth.json --apply-spec /root/ceph_cluster-spec-5-node-all-disks.yaml`

When boostrap completes it will print out the Console URL, the admin username, and the temporary admin password. The installation will continue for some time. Use this time to login to the Console and update the admin password. You can then watch the installation proceed in the console, or you can use the CLI:

`# ceph orch ps`

The cephFS protocol is not yet configured. first we add 2 Pools to build the cephFS service with.

`ceph osd pool create cephfs_data`  
`ceph osd pool create cephfs_metadata`  

Then we create cephFS.

`ceph fs new testme cephfs_metadata cephfs_data`

I chose to name my filesystem `testme`, however the name is arbitrary and you can choose whatever name suits you best.

There will be 4 RGWs once the cluster finishes installing. You will want to install a proxy and configure it to load balance object requests across the 4 RGW endpoints. That is left as an excersice for the reader.

When you look at ceph_cluster-spec-5-node-all-disks.yaml you will see that we have, per node on 5 nodes, 3 HDDs (sdb - sdd), one SSD which has the WAL and Rocks DB for the HDDs (sde), and one SSD we have created an OSD on (sdf). You can adapt this to your particular cluster, but do note that best practice is to always have your RGW index pool on flash media. If you do not then you will have essentially a write only cluster as indexing objects will be very very slow indeed.

After waiting for the cluster to finish installing, up to this point, we now need to move our RGW index onto the flash media. The method I will show is brute force, but works. First we need to determine the OSD names for the flash devices we will be using. In my case I have 5 nodes with 3 HDD OSDs per node that I added first, or 15 OSDs. Then I added the flash drives I will use for indexing last, so knowing that the OSDs are numbered starting at 0, the OSD names I am targeting are OSD.15 - OSD.19. So we update the CRUSH map by removing the device type **ssd** from them and replacing them with our arbitrary device type **index**.

`for i in {15..19}; do ceph osd crush rm-device-class osd.$i;done`  
`for i in {15..19}; do ceph osd crush set-device-class index osd.$i;done`

Now we create a new CRUSH rule for our index, `replicated_index`. This will be a replicated rule as that is best practice for index pools.

`ceph osd crush rule create-replicated replicated_index default host index`

Now create an object bucket (I use the Console) so that the RGW index pool will be created, `default.rgw.buckets.index`.

No update the index pool to use the new replicated_index CRUSH rule to place its data on the flash media we have configured for it.

`ceph osd pool set default.rgw.buckets.index crush_rule replicated_index`

## Disconnected Install (offline)
You will need to create a DNF/YUM repo to provide IBM Storage Ceph RPMs, and you will need to mirror the IBM Storage Ceph containers to a local registry.
