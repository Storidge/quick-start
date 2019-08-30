![Logo](https://i.imgur.com/pMGTbS7.png)

# Quick Start

Docker Swarm is a popular orchestration tool used for managing containerized applications. A Swarm cluster turns a group of Docker hosts into a single virtual system.

While containers provide a great develop and deploy applications efficiently, their transient nature mean containers lose all data when deleted. This is a problem for applications which need to persist data beyond the lifecycle of the container.

## Why Persistent Storage?

Persistent storage is a key requirement for virtually all enterprises, because most systems enterprises run require data that can be saved and tapped into later. These include systems which provide insights into consumer behavior and delivers actionable leads into what customers are looking for.

Persistent storage is desirable for container-based applications and Swarm clusters because data can be retained after applications running inside those containers are shut down. However, many deployments rely on external storage systems for data persistence.

In public cloud deployments, this means using managed services such as EBS, S3 and EFS. On-premise deployments typically use traditional NAS and SAN storage solutions which are cumbersome and expensive to operate.

## Deploy Swarm Cluster with Cloud Native Storage

Storidge’s CIO software was created to simplify the life of developers and operators. It is software defined storage designed to make cloud-native clusters and the applications and services running inside them more self-sufficient and portable, by providing highly available storage as a service.

This guide shows you how to easily deploy Storidge's Container IO (CIO) software. Follow the steps below to bring up a Swarm cluster with a Portainer dashboard, that's ready to run stateful apps in just a few minutes

Let’s get started!

## Prerequisites

First, you'll need to deploy the cluster resources to orchestrate:

- A minimum of three hosts (physical or virtual) are required to form a cluster. Four nodes minimum are recommended for production clusters. You can use `docker-machine create` to provision virtual hosts. Here are examples using [VirtualBox](https://rominirani.com/docker-swarm-tutorial-b67470cf8872) and [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-create-a-cluster-of-docker-containers-with-docker-swarm-and-digitalocean-on-centos-7).
- Each node will need a minimum of four drives; one boot drive and three data drives for CIO to ensure data redundancy and availability
- Configure networking to allow SSH connections across all hosts

## Step 1. Install cio software

Storidge's CIO software currently supports CentOS 7.6 (3.10 kernel), RHEL 7 (3.10 kernel) and Ubuntu 16.04LTS (4.4 kernel). Note that the desktop edition of Ubuntu 16.04 lists a 4.15 kernel which is not supported.

After verifying you have a supported distribution, run the convenience script below to begin installation.

`curl -fsSL ftp://download.storidge.com/pub/ce/cio-ce | sudo bash`

Example:
```
root@ip-172-31-27-160:~# curl -fsSL ftp://download.storidge.com/pub/ce/cio-ce | sudo bash
Started installing release 2879 at Sat Jul 13 02:52:57 UTC 2019
Loading cio software for: u16  (4.4.0-1087-aws)
Reading package lists... Done
Building dependency tree
.
.
.
Finished at Sat Jul 13 02:53:06 UTC 2019

Installation completed. cio requires a minimum of 3 local drives per node for data redundancy.

To start a cluster, run 'cioctl create' on primary node. To add a node, generate a join token
with 'cioctl join-token' on sds node. Then run the 'cioctl node add ...' output on this node.
```

**Install Additional Nodes**

You can add more nodes to the cluster to increase capacity, performance and enable high availability for your applications. Repeat the convenience script installation on all nodes that will be added to the cluster.

`curl -fsSL ftp://download.storidge.com/pub/ce/cio-ce | sudo bash`

Note: The use of convenience scripts is recommended for dev environments only, as root permissions are required to run them.

## Step 2. Configure cluster
With the CIO software installed on all nodes, the next step is to create a cluster and initialize it for use. As part of cluster creation, CIO will automatically discover and add drive resources from each node. Note that drives which are partitioned or have a file system will not be added to the storage pool.

Run the `cioctl create` command on the node you wish to be the leader of the cluster. This generates a `cioctl join` and a `cioctl init` command.

```
root@ip-172-31-22-159:~# cioctl create
Key Generation setup
Cluster started. The current node is now the primary controller node. To add a storage node to this cluster, run the following command:
    cioctl join 172.31.22.159 7598e9e2eb9fe221b98f9040cb3c73bc-bd987b6a

After adding all storage nodes, return to this node and run following command to initialize the cluster:
    cioctl init bd987b6a
```

Run the `cioctl join` command on nodes joining the cluster.

**Single node cluster**
To configure a single node cluster, just run `cioctl create --single-node` to create the cluster and automatically complete initialization.

## Step 3. Initialize cluster

With all nodes added, run the `cioctl init` command on the SDS node to start initializing the cluster.

```
root@ip-172-31-22-159:~# cioctl init bd987b6a
Configuring Docker Swarm cluster with Portainer service
<13>Jul 13 02:43:40 cluster: Setup AWS persistent hostname for sds node
<13>Jul 13 02:44:00 cluster: initialization started
<13>Jul 13 02:44:03 cluster: Setup AWS persistent hostnames
.
.
.
<13>Jul 13 02:47:24 cluster: Node initialization completed
<13>Jul 13 02:47:26 cluster: Start cio daemon
<13>Jul 13 02:47:34 cluster: Succeed: Add vd0: Type:2-copy, Size:20GB
<13>Jul 13 02:47:36 cluster: MongoDB ready
<13>Jul 13 02:47:37 cluster: Synchronizing VID files
<13>Jul 13 02:47:38 cluster: Starting API
```

**Note:**  For physical servers with SSDs, the initialization process will take about 30 minutes longer the first time. This extra time is used to characterize the available performance in the cluster. This performance information is used in CIO’s quality-of-service (QoS) feature to deliver guaranteed performance for individual applications.

## Login dashboard

At the end of initialization, you have a CIO storage cluster running. A Docker Swarm cluster will be automatically configured if one is not already running.

Run `docker node ls` to show the compute cluster nodes. Example:
```
root@ip-172-31-22-159:~# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
k9lwu33n1qvc4qw06t8rhmrn0     c-8ca106d7          Ready               Active              Reachable           18.09.6
18yxcme2s8b4q9jiq0iceh35y     c-6945fd81          Ready               Active              Leader              18.09.6
pmjn72izhrrb6hn2gnaq895c4 *   c-abc38f75          Ready               Active              Reachable           18.09.6
```

The Portainer service is launched at the end of initialization. Verify with `docker service ps portainer`:
```
root@ip-172-31-22-159:~# docker service ps portainer
ID                  NAME                IMAGE                        NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
zhu4ykc1hdlu        portainer.1         portainer/portainer:latest   c-6945fd81          Running             Running 4 minutes ago
```

Login to Portainer at any node's IP address on port 9000. Assign an admin password and you'll see the dashboard.

## Next steps

Refer to the [Getting Started guide](https://guide.storidge.com/) for exercises to create volumes, profiles, deploy stateful apps, test high availability, perform cluster management, etc.

Join us on slack @ http://storidge.com/join-cio-slack/ 
