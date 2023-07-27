# Install IBM Storage Ceph
This repository includes serveral ways to install IBM Storage Ceph including connected and disconnected options. This is not a comprehensive archive for every possible installation option, it is intended only as a short list of common install options that can be adapated to many IBM Storage Ceph installations.

First let's cover the difference between a connected and a disconnected install. A connected install means you have direct internet access to the IBM Storage Ceph RPMs and containers. A disconnected install means that the installation must use mirrored repositories for both RPMs and containers (registry).

## Connected Install

### Simple (All OSDs are identical)
This install assumes that all nodes are essentially identical, and all storage is both identical and are flash. An all HDD install is not recommended. There are two configuration files provided for this install option. The first is ceph_cluster-spec-4-node-all-disks.yaml and is for an object only cluster with 4 nodes. 4 Nodes is the minimum supported number of nodes for IBM Storage Ceph.

### Complex (OSDs are not identical or are compound)

## Disconnected Install (offline)
You will need to create a DNF/YUM repo to provide IBM Storage Ceph RPMs, and you will need to mirror the IBM Storage Ceph containers to a local registry.
