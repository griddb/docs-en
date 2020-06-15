# GridDB Features Reference								 

## Table of Contents
* [Introduction](#introduction)
* [Structure of GridDB](#structure-of-griddb)
* [GridDB data model](#data-model)
* [Database function](#database-function)
* [Admin function](#admin-function)
* [Parameter](#parameter)	


---							  
# Introduction									 

## Purpose of this documentation

This manual is targeted at designers and developers who perform system design and development using GridDB Community Edition.

The contents of this manual are as follows.

  - Structure of GridDB
      - Describes the cluster operating structure in GridDB.
  - The data model of GridDB
      - Describes the data model of GridDB.
  - Functions provided by GridDB
      - Describes the data management functions provided by GridDB.
  - Parameter
      - Describes the parameters to control the operations in GridDB.

#### :warning: Note
- GridDB is a database that manages a group of data (known as a row) that is made up of a key and multiple values. Besides having a composition of an in-memory database that arranges all the data in the memory, it can also adopt a hybrid composition combining the use of a disk (including SSD as well) and a memory. 
- OS user (gsadm) is created when GridDB is installed using the package.


## Terminology

Describes the terms used in GridDB in a list.

| Term                                                | Description                                                                                                                                                                                                                                                                                                                                                                                  |
|--------------------------------|------------------------------------------------------------------------|
| Node                                                | Refers to the individual server process to perform data management in GridDB.                                                                                                                                                                                                                                                                                                                |
| Cluster                                             | Single or a set of nodes that perform data management together in an integrated manner.                                                                                                                                                                                                                                                                                                      |
| Master node                                         | Node to perform a cluster management process.                                                                                                                                                                                                                                                                                                                                                |
| Follower node                                       | All other nodes in the cluster other than the master node.                                                                                                                                                                                                                                                                                                                                   |
| number of nodes constituting a cluster              | Refers to the number of nodes constituting a GridDB cluster. When starting GridDB for the first time, the number is used as a threshold value for the cluster to be valid. (Cluster service is started when the number of nodes constituting a cluster joins the cluster.)                                                                                                                   |
| number of nodes already participating in a cluster  | Number of nodes currently in operation that have been incorporated into the cluster among the nodes constituting the GridDB cluster.                                                                                                                                                                                                                                                         |
| Block                                               | A block is a data unit for data persistence processing in a disk (hereinafter referred to a checkpoint) and is the smallest physical data management unit in GridDB. Multiple container data are arranged in a block. Block size is set up in a definition file (cluster definition file) before the initial startup of GridDB.                                                              |
| Partitioned table                                   | Data management unit to arrange a container. Smallest data arrangement unit among clusters, and data movement and replication unit for adjusting the load balance between nodes (rebalancing) and for managing data replicas in case of failure.                                                                                                                                             |
| Partition group                                     | A group summarizing multiple partitions which is equivalent to the data file in the file system when the data is perpetuated in a disk. 1 checkpoint file corresponds to 1 partition group. Partition groups are created according to the number of concurrency (/dataStore/concurrency) in the node definition file.                                                                        |
| Row                                                 | Refers to one row of data registered in a container or table. Multiple rows are registered in a container or table. A row consists of values of columns corresponding to the schema definition of the container (table).                                                                                                                                                                     |
| Container (Table)                                   | Container to manage a set of rows. It may be called a container when operated with NoSQL I/F, and may be called a table when operated with NewSQL I/F. What these names refer are the same object, only in different names. A container has two data types: collection and timeseries container.                                                                                             |
| Collection (table)                                  | One type of container (table) to manage rows having a general key.                                                                                                                                                                                                                                                                                                                           |
| Timeseries container (timeseries table)             | One type of container (table) to manage rows having a timeseries key. Possesses a special function to handle timeseries data.                                                                                                                                                                                                                                                                |
| Database file                                       | A database file is a file group consisting of transaction log file and checkpoint file that are perpetuated to a HDD or SSD. Transaction log file is updated every time the GridDB database is updated or a transaction occurs, whereas the checkpoint file is written at a specified time interval.                                                                                         |
| Checkpoint file                                     | A file written into a disk by a partition group. Updated information is reflected in the memory by a cycle of the node definition file (/checkpoint/checkpointInterval).                                                                                                                                                                                                                     |
| Transaction log file                                | Update information of the transaction is saved sequentially as a log.                                                                                                                                                                                                                                                                                                                        |
| LSN (Log Sequence Number)                           | Shows the update log sequence number, which is assigned to each partition during the update in a transaction. The master node of a cluster configuration maintains the maximum number of LSN (MAXLSN) of all the partitions maintained by each node.                                                                                                                                         |
| Replica                                             | Replication is the process of creating an exact copy of the original data. In this case, one or more replica are created and stored on multiple nodes, which results to the creation of partition across the nodes. There are 2 forms of replica, master and backup. The former one refers to the original or master data, whereas the latter one is used in case of failure as a reference. |
| Owner node                                          | A node that can update a container in a partition. A node that records the container serving as a master among the replicated containers.                                                                                                                                                                                                                                                    |
| Backup node                                         | A node that records the container for backup data among the replicated containers.                                                                                                                                                                                                                                                                                                           |
| Definition file                                     | Definition file includes two types of parameter files: gs_cluster.json, hereinafter referred to as a cluster definition file, used when composing a cluster; gs_node.json, hereinafter referred to as a node definition file, used to set the operations and resources of the node in a cluster. It also includes a user definition file for GridDB administrator users.                   |
| Event log file                                      | Event logs of the GridDB server are saved in this file including messages such as errors, warnings and so on.                                                                                                                                                                                                                                                                                |
| OS user (gsadm)                                     | An OS user has the right to execute operating functions in GridDB. An OS user named gsadm is created during the GridDB installation.                                                                                                                                                                                                                                                         |
| Administrator user                                  | An administrator user is a GridDB user prepared to perform operations in GridDB.                                                                                                                                                                                                                                                                                                             |
| General user                                        | A user used in the application system.                                                                                                                                                                                                                                                                                                                                                       |
| user definition file                                | File in which an administrator user is registered. During initial installation, 2 administrators, system and admin, are registered.                                                                                                                                                                                                                                                          |
| Cluster database                                    | General term for all databases that can be accessed in a GridDB cluster system.                                                                                                                                                                                                                                                                                                              |
| Database                                            | Theoretical data management unit created in a cluster database. A public database is created in a cluster database by default. Data separation can be realized for each user by creating a new database and giving a general user the right to use it.                                                                                                                                       |
| Failover                                            | When a failure occurs in a cluster currently in operation, the structure allows the backup node to automatically take over the function and continue with the processing.                                                                                                                                                                                                                    |
| Client failover                                     | When a failure occurs in a cluster currently in operation, the structure allows the backup node to be automatically re-connected to continue with the processing as a retry process when a failure occurs in the API on the client side.                                                                                                                                                     |
| Table partitioning                                  | Function to access a huge table quickly by allowing concurrent execution by processors of multiple nodes, and the memory of multiple nodes to be used effectively by distributing the placement of a large amount of table data with multiple data registrations in multiple nodes.                                                                                                          |
| Data partition                                      | General name of data storage divided by table partitioning. Multiple data partitions are created for a table by table partitioning. Data partitions are distributed to the nodes like normal containers. The number of data partitions and the range of data stored in each data partition are depending on the type of table partitioning (hash, interval or interval-hash).                |
| Data Affinity                                       | A function to raise the memory hit rate by placing highly correlated data in a container in the same block and localizing data access.                                                                                                                                                                                                                                                       |
| Placement of container/table based on node affinity | A function to reduce the network load during data access by placing highly correlated containers in the same node.                                                                                                                                                                                                                                                                           |

----
# Structure of GridDB

Describes the cluster operating structure in GridDB.
Note that **GridDB Community Edition operates on a single node configuration, and does not support multiple nodes cluster configuration**.


## Composition of a cluster

GridDB is operated by clusters which are composed of multiple nodes. Before accessing the database from an application system, nodes must be started and the cluster must be constituted, that is, cluster service is executed.

A cluster is formed and cluster service is started when a number of nodes specified by the user joins the cluster. Cluster service will not be started and access from the application will not be possible until all nodes constituting a cluster have joined the cluster.

A cluster needs to be constituted even when operating GridDB with a single node. In this case, the number of nodes constituting a cluster is 1. A composition that operates a single node is known as a single composition.

![Cluster name and number of nodes constituting a cluster](img/arc_clusterNameCount.png)




A cluster name is used to distinguish a cluster from other clusters so as to compose a cluster using the right nodes selected from multiple GridDB nodes on a network. Using cluster names, multiple GridDB clusters can be composed in the same network. 
A cluster is composed of nodes with the following features in common: cluster name, the number of nodes constituting a cluster, and the connection method setting. A cluster name needs to be set in the cluster definition file for each node constituting a cluster, and needs to be specified as a parameter when composing a cluster as well.

The method of constituting a cluster using multicast is called multicast method. See [Cluster configuration methods](#cluster_configuration_methods) for details.

The operation of a cluster composition is shown below.

![Operation of a cluster composition](img/arc_clusterConfigration.png)



To start up a node and compose a cluster, the [operation commands](#operating_commands) gs_startnode/gs_joincluster command are used. In addition, there is a service control function to start up the nodes at the same time as the OS and to compose the cluster.


To compose a cluster, the number of nodes joining a cluster (number of nodes constituting a cluster) and the cluster name must be the same for all the nodes joining the cluster.

Even if a node fails and is separated from the cluster after operation in the cluster started, cluster service will continue so long as the majority of the number of nodes is joining the cluster.

Since cluster operation will continue as long as the majority of the number of nodes is in operation. So, a node can be separated from the cluster for maintenance while keeping the cluster in operation. The node can be get back into the cluster via network after the maintenance. Nodes can also be added via network to reinforce the system.

The following two networks can be separated: the network that communicates within the cluster and the network dedicated to client communication. 

### Status of node

Nodes have several types of status that represent their status. The status changes by user command execution or internal processing of the node. The [status of a cluster](#status_of_cluster) is determined by the status of the nodes in a cluster.

This section explains types of node status, status transition, and how to check the node status.

#### Types of node status
    
| Node status | Description                                                                                                                                                                                                                                                                                                                                                                                                     |
-----------|----------------------------------------------|
| STOP        | The GridDB server has not been started in the node.                                                                                                                                                                                                                                                                                                                                                             |
| STARTING    | The GridDB server is starting in the node. Depending on the previous operating state, start-up processes such as recovery processing of the database are carried out. The only possible access from a client is checking the status of the system with a gs_stat command. Access from the application is not possible.                                                                       |
| STARTED     | The GridDB server has been started in the node. However, access from the application is not possible as the node has not joined the cluster. To obtain the cluster composition, execute a cluster operating command, such as gs_joincluster to join the node to the cluster.                                                                                                                         |
| WAIT        | The system is waiting for the cluster to be composed. Nodes have been informed to join a cluster but the number of nodes constituting a cluster is insufficient, so the system is waiting for the number of nodes constituting a cluster to be reached. WAIT status also indicates the node status when the number of nodes constituting a cluster drops below the majority and the cluster service is stopped. |
| SERVICING   | A cluster has been constituted and access from the application is possible. However, access may be delayed if synchronization between the clusters of the partition occurs due to a re-start after a failure when the node is stopped or the like.                                                                                                                                                              |
| STOPPING    | Intermediate state in which a node has been instructed to stop but has not stopped yet.                                                                                                                                                                                                                                                                                                                         |
| ABNORMAL    | The state in which an error is detected by the node in SERVICING state or during state transition. A node in the ABNORMAL state will be automatically separated from the cluster. After collecting system operation information, it is necessary to forcibly stop and restart the node in the ABNORMAL state. By re-starting the system, recovery processing will be automatically carried out.                 |
    
#### Transition in the node status
    
![Node status](img/arc_NodeStatus.png)


| State transition | State transition event | Description                                                                                                                                                                                                                                             |
| ---------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ①                | Command execution      | Start a node by executing the commands such as gs_startnode command.                                                                                                                                                     |
| ②                | System                 | Status changes automatically at the end of recovery processing or loading of database files.                                                                                                                                                            |
| ③                | Command execution      | Joining a node to a cluster by executing the commands such as gs_joincluster command, and service start-up.                                                                                                                  |
| ④                | System                 | Status changes automatically when the required number of component nodes join a cluster.                                                                                                                                                                |
| ⑤                | System                 | Status changes automatically when some nodes consisting the cluster are detached from the service due to a failure or by some other reasons, and the number of nodes joining the cluster become less than half of the value set in the definition file. |
| ⑥                | Command execution      | Detaches a node from a cluster by executing the commands such as gs_leavecluster command.                                                                                                                                                    |
| ⑦                | Command execution      | Detaches a node from a cluster by executing the commands such as gs_leavecluster/gs_stopcluster command.                                                                                                                                    |
| ⑧                | Command execution      | Stop a node by executing the commands such as gs_stopnode command.                                                                                                                                                           |
| ⑨                | System                 | Stops the server process once the final processing ends                                                                                                                                                                                                 |
| ⑩                | System                 | Detached state due to a system failure. In this state, the node needs to be stopped by force once.                                                                                                                                                      |
    
	
##### How to check the node status
    
The node status is determined by the combination of the node status and the node role.

The operation status of a node and the role of a node can be checked from the result of the gs_stat command, which is in json format. That is, for the operation status of a node, check the value of /cluster/nodeStatus, for the role of a node, check /cluster/clusterStatus)

The table below shows the node status, determined by the combination of the operation status of a node and the role of a node.
    
| Node status | Operation status of a node<br>(/cluster/nodeStatus)  | Role of a node<br>(/cluster/clusterStatus) |
|------------|------------------------------|------------------------|
| STOP       | －(Connection error of gs_stat)       | －(Connection error of gs_stat)    |
| STARTING   | INACTIVE                     | SUB_CLUSTER            |
| STARTED    | INACTIVE                     | SUB_CLUSTER            |
| WAIT       | ACTIVE                       | SUB_CLUSTER           |
| SERVICING  | ACTIVE                       | MASTER or FOLLOWER   |
| STOPPING   | NORMAL_SHUTDOWN              | SUB_CLUSTER           |
| ABNORMAL   | ABNORMAL                     | SUB_CLUSTER           |
	
#### Operation status of a node
        
The table below shows the operation status of a node. Each state is expressed as the value of /cluster/nodeStatus of the gs_stat command.
        
| Operation status of a node | Description                          |
| -------------------------- | ------------------------------------ |
| ACTIVE                     | Active state                         |
| ACTIVATING                 | In transition to an active state     |
| INACTIVE                   | Non-active state                     |
| DEACTIVATING               | In transition to a non-active state. |
| NORMAL_SHUTDOWN            | Under shutdown process               |
| ABNORMAL                   | Abnormal state                       |
        

#### Role of a node
        
The table below shows the role of a node. Each state is expressed as the value of /cluster/clusterStatus of the gs_stat command.
        
A node has two types of roles: "master" and "follower". To start a cluster, one of the nodes which constitute the cluster needs to be a "master." The master manages the whole cluster. All the nodes other than the master become "followers." A follower performs cluster processes, such as a synchronization, following the directions from the master.
        
| Role of a node           | Description    |
| ------------------------ | -------------- |
| MASTER                   | Master         |
| FOLLOWER                 | Follower       |
| SUB_CLUSTER/SUB_MASTER   | Role undefined |
        

<a id="status_of_cluster"></a>
### Status of cluster

The cluster operating status is determined by the state of each node, and the status may be one of 3 states - IN OPERATION/INTERRUPTED/STOPPED.

During the initial system construction, cluster service starts after all the nodes, the number of which was specified by the user as the number of nodes constituting a cluster, have joined the cluster.

During initial cluster construction, the state in which the cluster is waiting to be composed when all the nodes that make up the cluster have not been incorporated into the cluster is known as [INIT_WAIT]. When the number of nodes constituting a cluster has joined the cluster, the state will automatically change to the operating state.

Operation status includes two states, [STABLE] and [UNSTABLE].

  - [STABLE] state
      - State in which a cluster has been formed by the number of nodes specified in the number of nodes constituting a cluster and service can be provided in a stable manner.
  - [UNSTABLE] state
      - A cluster in this state is joined by the nodes less than "the number of the nodes constituting the cluster" but more than half the constituting clusters are in operation.
      - Cluster service will continue for as long as a majority of the number of nodes constituting a cluster is in operation.

A cluster can be operated in an [UNSTABLE] state as long as a majority of the nodes are in operation even if some nodes are detached from a cluster due to maintenance and for other reasons.

Cluster service is interrupted automatically in order to avoid a split brain when the number of nodes constituting a cluster is less than half the number of nodes constituting a cluster. The status of the cluster will become [WAIT].

  - What is split brain?
    
    A split brain is an action where multiple cluster systems performing the same process provide simultaneous service when a system is divided due to a hardware or network failure in a tightly-coupled system that works like a single server interconnecting multiple nodes. If the operation is continued in this state, data saved as replicas in multiple clusters will be treated as master data, causing data inconsistency.

To resume the cluster service from a [WAIT] state, add the node, which recovered from the abnormal state, or add a new node, by using a node addition operation. After the cluster is joined by all the nodes, the number of which is the same as the one specified in "the number of nodes constituting a cluster", the status will be [STABLE], and the service will be resumed.

Even when the cluster service is disrupted, since the number of nodes constituting a cluster becomes less than half due to failures in the nodes constituting the cluster, the cluster service will be automatically restarted once a majority of the nodes joine the cluster by adding new nodes and/or the nodes restored from the errors to the cluster.

![Cluster status](img/arc_clusterStatus.png)


A STABLE state is a state in which the value of the json parameter shown in gs_stat, /cluster/activeCount, is equal to the value of /cluster/designatedCount.

```
$ gs_stat -u admin/admin
{
    "checkpoint": {
        "archiveLog": 0,
　　　　　：
　　　　　：
    },
    "cluster": {
        "activeCount":4,　　　　　　　　　　　 // Nodes in operation within the cluster
        "clusterName": "test-cluster",
        "clusterStatus": "MASTER",
        "designatedCount": 4,                  // Number of nodes constituting a cluster
        "loadBalancer": "ACTIVE",
        "master": {
            "address": "192.168.0.1",
            "port": 10040
        },
        "nodeList": [　　　　　　　　　　　　　// Node list constituting a cluster
            {
                "address": "192.168.0.1",
                "port": 10040
            },
            {
                "address": "192.168.0.2",
                "port": 10040
            },
            {
                "address": "192.168.0.3",
                "port": 10040
            },
            {
                "address": "192.168.0.4",
                "port": 10040
            },

        ],
        ：
        ：
```

### Status of partition

The partition status represents the status of the entire partition in a cluster, showing whether the partitions in an operating cluster are accessible, or the partitions are balanced.


| Partition status | Description |
|--------------|----------------|
| NORMAL       | All the partitions are in normal states where all of them are placed as planned. |
| NOT_BALANCE  | With no replica_loss, no owner_loss but partition placement is unbalanced. |
| REPLICA_LOSS | Replica data is missing in some partitions.<br /> (Availability of the partition is reduced, that is, the node cannot be detached from the cluster.) |
| OWNER_LOSS   | Owner data is missing in some partitions.<br /> (The data of the partition are not accessible.)      |
| INITIAL      | The initial state no partition has joined the cluster |

Partition status can be checked by executing gs_stat command to a master node. (The state is expressed as the value of /cluster/partitionStatus)

```
$ gs_stat -u admin/admin
{
　　：
　　：
"cluster": {
    ：
    "nodeStatus": "ACTIVE",
    "notificationMode": "MULTICAST",
    "partitionStatus": "NORMAL",
    ：
```

[Notes]
  - The value of /cluster/partitionStatus of the nodes other than a master node may not be correct. Be sure to check the value of a master node.

<a id="cluster_configuration_methods"></a>

## Cluster configuration methods

A cluster consists of one or more nodes connected in a network. Each node maintains a list of the other nodes' addresses for  communication purposes.

GridDB supports 3 cluster configuration methods for configuring the address list. Different cluster configuration methods can be used depending on the environment or use case. Connection method of client or operational tool may also be different depending on the configuration methods.

Three cluster configuration methods are available: Multicast method, Fixed list method and Provider method. Multicast method is recommended.

Fixed list or provider method can be used in the environment where multicast is not supported.

  - Multicast method
      - This method performs node discovery in multi-cast to automatically configure the address list.
  - Fixed list method
      - A fixed address list is saved in the cluster definition file.
  - Provider method
      - Provider method
      - The address provider can be configured as a Web service or as a static content.

The table below compares the three cluster configuration methods.

| Property         | Multicast method (recommended)             | Fixed list method                                 | Provider method          |
|--------------|------------------------------------|-----------------------------------------------|-----------------------|
| Parameters         | - Multicast address and port      | - List of IP address and port of all the node           | - URL of the address provider        |
| Use case   | - When multicast is supported          | - When multicast is not supported<br /> - System scale estimation can be performed accurately | - When multicast is not supported<br /> - System scale estimation can not be performed        |
| Cluster operation | - Perform automatic discovery of nodes at a specified time interval | - Set a common address list for all nodes<br /> - Read that list only once at node startup | - Obtain the address list at a specified time interval from address provider |
| Pros.     | - No need to restart the cluster when adding nodes      | - No mistake of configuration by consistency check of the list | - No need to restart the cluster when adding nodes    |
| Cons.   | - Multicast is required for client connection   | - Need to restart cluster when adding nodes<br /> - Need to update the connection setting of the client | - Need to ensure the availability of the address provider     |


### Setting up cluster configuration files

Fixed list method or provider method can be used in the environment where multicast is not supported. Network setting of fixed list method and provider method is as follows.

#### FIXED_LIST: fixed list method

When a fixed address list is given to start a node, the list is used to compose the cluster.

When composing a cluster using the fixed list method, configure the parameters in the cluster definition file.

**cluster definition file**

| Property                    | JSON Data type | Description                                                                                    |
| --------------------------- | -------------- | ---------------------------------------------------------------------------------------------- |
| /cluster/notificationMember | string         | Specify the address list when using the fixed list method as the cluster configuration method. |

A configuration example of a cluster definition file is shown below.

```
{
                             :
                             :
    "cluster":{
        "clusterName":"yourClusterName",
        "replicationNum":2,
        "heartbeatInterval":"5s",
        "loadbalanceCheckInterval":"180s",
        "notificationMember": [
            {
                "cluster": {"address":"172.17.0.44", "port":10010},
                "sync": {"address":"172.17.0.44", "port":10020},
                "system": {"address":"172.17.0.44", "port":10040},
                "transaction": {"address":"172.17.0.44", "port":10001},
                "sql": {"address":"172.17.0.44", "port":20001}
            },
            {
                "cluster": {"address":"172.17.0.45", "port":10010},
                "sync": {"address":"172.17.0.45", "port":10020},
                "system": {"address":"172.17.0.45", "port":10040},
                "transaction": {"address":"172.17.0.45", "port":10001},
                "sql": {"address":"172.17.0.45", "port":20001}
            },
            {
                "cluster": {"address":"172.17.0.46", "port":10010},
                "sync": {"address":"172.17.0.46", "port":10020},
                "system": {"address":"172.17.0.46", "port":10040},
                "transaction": {"address":"172.17.0.46", "port":10001},
                "sql": {"address":"172.17.0.46", "port":20001}
            }
        ]
    },
                             :
                             :
}
```

#### PROVIDER: provider method

Get the address list supplied by the address provider to perform cluster configuration.

When composing a cluster using the provider method, configure the parameters in the cluster definition file.

**cluster definition file**

| Property                                     | JSON Data type | Description                                                                                                                                |
| -------------------------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| /cluster/notificationProvider/url            | string         | Specify the URL of the address provider when using the provider method as the cluster configuration method.                                |
| /cluster/notificationProvider/updateInterval | string         | Specify the interval to get the list from the address provider. Specify the value more than 1 second and less than 2<sup>31</sup> seconds. |

A configuration example of a cluster definition file is shown below.

```
{
                             :
                             :
    "cluster":{
        "clusterName":"yourClusterName",
        "replicationNum":2,
        "heartbeatInterval":"5s",
        "loadbalanceCheckInterval":"180s",
        "notificationProvider":{
            "url":"http://example.com/notification/provider",
            "updateInterval":"30s"
        }
    },
                             :
                             :
}
```

The address provider can be configured as a Web service or as a static content. The address provider needs to provide the following specifications.

  - Compatible with the GET method.
  - When accessing the URL, the node address list of the cluster containing the cluster definition file in which the URL is written is returned as a response.
      - Response body: Same JSON as the contents of the node list specified in the fixed list method
      - Response header: Including Content-Type:application/json

An example of a response sent from the address provider is as follows.

```
$ curl http://example.com/notification/provider
[
    {
        "cluster": {"address":"172.17.0.44", "port":10010},
        "sync": {"address":"172.17.0.44", "port":10020},
        "system": {"address":"172.17.0.44", "port":10040},
        "transaction": {"address":"172.17.0.44", "port":10001},
        "sql": {"address":"172.17.0.44", "port":20001}
    },
    {
        "cluster": {"address":"172.17.0.45", "port":10010},
        "sync": {"address":"172.17.0.45", "port":10020},
        "system": {"address":"172.17.0.45", "port":10040},
        "transaction": {"address":"172.17.0.45", "port":10001},
        "sql": {"address":"172.17.0.45", "port":20001}
    },
    {
        "cluster": {"address":"172.17.0.46", "port":10010},
        "sync": {"address":"172.17.0.46", "port":10020},
        "system": {"address":"172.17.0.46", "port":10040},
        "transaction": {"address":"172.17.0.46", "port":10001},
        "sql": {"address":"172.17.0.46", "port":20001}
    }
]
```

[Note]
  - Specify the serviceAddress and servicePort of the node definition file in each module (cluster,sync etc.) for each address and port.
  - Set either the /cluster/notificationAddress, /cluster/notificationMember, /cluster/notificationProvider in the cluster definition file to match the cluster configuration method used.



----

# Data model

GridDB is a unique Key-Container data model that resembles Key-Value. It has the following features.
  - A concept resembling a RDB table that is a container for grouping Key-Value has been introduced.
  - A schema to define the data type for the container can be set. An index can be set in a column.
  - Transactions can be carried out on a row basis within the container. In addition, ACID is guaranteed on a container basis.


![Data model](img/arc_DataModel.png)


GridDB manages data on a block, container, table, row, partition, and partition group basis.


  - Block
    
    A block is a data unit for data persistence processing in a disk (hereinafter referred to a checkpoint) and is the smallest physical data management unit in GridDB. Multiple container data are arranged in a block. 
	Block size is set up in a definition file (cluster definition file) before the initial startup of GridDB.
    
    As a database file is created during initial startup of the system, the block size cannot be changed after initial startup of GridDB.

  - Container (Table)
    
    A container is a data structure that serves as an interface with the user. A container consists of multiple blocks. Data structure serving as an I/F with the user. Container to manage a set of rows. It is called a container when operating with NoSQL I/F, and a table when operating with NewSQL I/F. 2 data types exist, collection (table) and timeseries container (timeseries table).
    
    Before registering data in an application, there is a need to make sure that a container (table) is created beforehand. Data is registered in a container (table).

  - Row
    
    A row refers to a row of data to be registered in a container or table. Multiple rows can be registered in a container or table but this does not mean that data is arranged in the same block. Depending on the registration and update timing, data is arranged in suitable blocks within partitions.
    
    A row includes columns of more than one data type.

  - Partitioned table
    
    A partition is a data management unit that includes 1 or more containers or tables.
    
    A partition is a data arrangement unit between clusters for  managing the data movement to adjust the load balance between nodes and data multiplexing (replica) in case of a failure. Data replica is arranged in a node to compose a cluster on a partition basis.
    
    A node that can update a container in a partition is called an owner node and one owner node is allocated to one partition. A node that maintains replicas other than owner nodes is a backup node. Master data and multiple backup data exist in a partition, depending on the number of replicas set.
    
    The relationship between a container and a partition is persistent and the partition which has a specific container is not changed. The relationship between a partition and a node is temporary and the autonomous data placement may cause partition migration to another node.

  - Partition group
    
    A group of multiple partitions is known as a partition group.
    
    Data maintained by a partition group is saved in an OS disk as a physical database file. A partition group is created with a number that depends on the degree of parallelism of the database processing threads executed by the node.


![Data management unit](img/arc_DataPieces.png)

　


<a id="label_container"></a>
## Container

To register and search for data in GridDB, a container (table) needs to be created to store the data. Data structure serving as an I/F with the user. Container to manage a set of rows. It is called a container when operating with NoSQL I/F, and a table when operating with NewSQL I/F.

The naming rules for containers (tables) are the same as those for databases.
  - A string consisting of alphanumeric characters, the underscore mark, the hyphen mark, the dot mark, the slash mark and the equal mark can be specified. The container name should not start with a number.
  - Although the name is case sensitive, a container (table) cannot be created if it has the same name as an existing container when they are case insensitive.


[Notes]
  - Avoid the name already used for naming a [view](#label_view) in the same database.



### Type

There are 2 container (table) data types. 
A **timeseries container (timeseries table)** is a data type which is suitable for managing hourly data together with the occurrence time while a **collection (table)** is suitable for managing a variety of data.


### Data type

The schema can be set in a container (table). The basic data types that can be registered in a container (table) are the basic data type and array data type .

#### Basic data types

Describes the basic data types that can be registered in a container (table). A basic data type cannot be expressed by a combination of other data types.

| JSON Data type | Description                                                                                                           |
| -------------- | --------------------------------------------------------------------------------------------------------------------- |
| BOOL           | True or false                                                                                                         |
| STRING         | Composed of an arbitrary number of characters using the unicode code point                                            |
| BYTE           | Integer value from -2<sup>7</sup>to 2<sup>7</sup>-1 (8bits)                                                           |
| SHORT          | Integer value from -2<sup>15</sup>to 2<sup>15</sup>-1 (16bits)                                                        |
| INTEGER        | Integer value from -2<sup>31</sup>to 2<sup>31</sup>-1 (32bits)                                                        |
| LONG           | Integer value from -2<sup>63</sup>to 2<sup>63</sup>-1 (64bits)                                                        |
| FLOAT          | Single precision (32 bits) floating point number defined in IEEE754                                                   |
| DOUBLE         | Double precision (64 bits) floating point number defined in IEEE754                                                   |
| TIMESTAMP      | Data type expressing the date and time Data format maintained in the database is UTC, and accuracy is in milliseconds |
| GEOMETRY       | Data type to represent a space structure                                                                              |
| BLOB           | Data type for binary data such as images, audio, etc.                                                                 |

The following restrictions apply to the size of the data that can be managed for STRING, GEOMETRY and BLOB data. The restriction value varies according to the block size which is the input/output unit of the database in the GridDB definition file (gs_node.json).

| Data type | Block size (64KB)                                        | Block size (from 1MB to 32MB)                             |
| --------- | -------------------------------------------------------- | --------------------------------------------------------- |
| STRING    | Maximum 31KB (equivalent to UTF-8 encode)                | Maximum 128KB (equivalent to UTF-8 encode)                |
| GEOMETRY  | Maximum 31KB (equivalent to the internal storage format) | Maximum 128KB (equivalent to the internal storage format) |
| BLOB      | Maximum 1GB - 1Byte                                      | Maximum 1GB - 1Byte                                       |


**GEOMETRY-type (Spatial-type)**

GEOMETRY-type (Spatial-type) data is often used in map information system and available only for a NoSQL interface, not supported by a NewSQL interface.

GEOMETRY type data is described using WKT (Well-known text). WKT is formulated by the Open Geospatial Consortium (OGC), a nonprofit organization promoting standardization of information on geospatial information. In GridDB, the spatial information described by WKT can be stored in a column by setting the column of a container as a GEOMETRY type.

GEOMETRY type supports the following WKT forms.

- POINT
  - Point represented by two or three-dimensional coordinate.
  - Example) POINT(0 10 10)
- LINESTRING
  - Set of straight lines in two or three-dimensional space    represented by two or more points.
  - Example) LINESTRING(0 10 10, 10 10 10, 10 10 0)
- POLYGON
  - Closed area in two or three-dimensional space represented by a set of straight lines. Specify the corners of a POLYGON counterclockwise. When building an island in a POLYGON, specify internal points clockwise.
  - Example) POLYGON((0 0,10 0,10 10,0 10,0 0)), POLYGON((35 10, 45 45, 15 40, 10 20, 35 10),(20 30, 35 35, 30 20, 20 30))
- POLYHEDRALSURFACE
  - Area in the three-dimensional space represented by a set of the specified area.
  - Example) POLYHEDRALSURFACE(((0 0 0, 0 1 0, 1 1 0, 1 0 0, 0 0 0)), ((0 0 0, 0 1 0, 0 1 1, 0 0 1, 0 0 0)), ((0 0 0, 1 0 0, 1 0 1, 0 0 1, 0 0 0)), ((1 1 1, 1 0 1, 0 0 1, 0 1 1, 1 1 1)), ((1 1 1, 1 0 1, 1 0 0, 1 1 0, 1 1 1)), ((1 1 1, 1 1 0, 0 1 0, 0 1 1, 1 1 1)))
- QUADRATICSURFACE
  - Two-dimensional curved surface in a three-dimensional space represented by defining equation f(X) = <AX, X> + BX + c.

The space structure written by QUADRATICSURFACE cannot be stored in a container, only can be specified as a search condition.

Operations using GEOMETRY can be executed with API or TQL.

With TQL, management of two or three-dimensional spatial structure is possible. Generating and judgement function are also provided.

```
 SELECT * WHERE ST_MBRIntersects(geom, ST_GeomFromText('POLYGON((0 0,10 0,10 10,0 10,0 0))'))
```

See "GridDB TQL Reference" ([GridDB_TQL_Reference](GridDB_TQL_Reference.md)) for details of the functions of TQL.

#### Hybrid types

A data type composed of a combination of basic data types that can be registered in a container. The only hybrid data type in the current version is an array.

  - Array
    
    Expresses an array of values. Among the basic data types, only GEOMETRY and BLOB data cannot be maintained as an array. The restriction on the data volume that can be maintained in an array varies according to the block size of the database.
    
    | Data type        | Block size (64KB) | Block size (from 1MB to 32MB) |
    | ---------------- | ----------------- | ----------------------------- |
    | Number of arrays | 4000              | 65000                         |
    
[Note]
The following restrictions apply to TQL operations in an array column.
  - Although the i-th value in the array column can be compared,  calculations (aggregation) cannot be performed on all the elements.
  - (Example) When columnA was defined as an array
      - The elements in an array such as select * where ELEMENT (0, column A) > 0 can be specified and compared. However, a variable cannot be specified instead of "0" in the ELEMENT.
      - Aggregation such as select SUM (column A) cannot be carried out.


<a id="primary_key"></a>
### Primary key

A ROWKEY key can be set in a container (table), The uniqueness of a row with a set ROWKEY is guaranteed. NULL is not allowed in the column ROWKEY is set.

In NewSQL I/F, ROWKEY is called as PRIMARY KEY.

  - For a timeseries container (timeseries table)
      - A ROWKEY can be set in the first column of the row. (This is set in Column No. 0 since columns start from 0 in GridDB.)
      - ROWKEY (PRIMARY KEY) is a TIMESTAMP
      - Must be specified.
  - For a collection (table)
      - ROWKEY (PRIMARY KEY) can be set to multiple columns that are continuous from the first column. The ROWKEY set to multiple columns is called composite ROWKEY, which can be set up to 16 columns.
          - Example) ROWKEY can be set to str1, str2, str3, which are consecutive from the first column.
			```
            CREATE TABLE sample_table1 
			(str1 string, str2  string, str3 string, str4 string, str5 string, int1 integer, 
			PRIMARY KEY(str1, str2, str3));
            ```
          - Example) ROWKEY can not be set to str1, str3, str4, which are not consecutive columns. Executing the following SQL will cause an error.
            ```
            CREATE TABLE sample_table2  
			(str1 string, str2 string, str3 string, str4 string, str5 string, int1 integer, 
			PRIMARY KEY(str1, str3, str4));
			```
      - A ROWKEY (PRIMARY KEY) is either a STRING, INTEGER, LONG or TIMESTAMP column.
      - Need not be specified.

A default index prescribed in advance according to the column data type can be set in a column set in ROWKEY (PRIMARY KEY).

In the current version GridDB, the default index of all STRING, INTEGER, LONG or TIMESTAMP data that can be specified in a ROWKEY (PRIMARY KEY) is the TREE index.


<a id="label_view"></a>
## View

View provides reference to data in a container.

Define a reference (SELECT statement) to a container when creating a view. A view is an object similar to a container, but it does not have real data. When executing a query containing a view, the SELECT statement, which was defined when the view was created, is evaluated, and a result is returned.

Views can only be referenced (SELECT), neither adding data (INSERT), updating (UPDATE), nor deletion data (DELETE) are not accepted.


[Notes]
  - Avoid the name already used for naming a container in the same  database.
  - The naming rule of a view is the same as the [naming rule of a  container](#label_container).

　　
---

# Database function

## Resource management

Besides the database residing in the memory, other resources constituting a GridDB cluster are perpetuated to a disk. The perpetuated resources are listed below.

  - Database file
    
    A database file is a file group consisting of transaction log file and checkpoint file that are perpetuated to a HDD or SSD.  Transaction log file is updated every time the GridDB database is updated or a transaction occurs, whereas the checkpoint file is written at a specified time interval.

  - Checkpoint file
    
    A checkpoint file is the perpetuation of a partition group data from the memory to the disk at a specified time interval, Updated
    information is reflected in the memory by a cycle of the node  definition file (/checkpoint/checkpointInterval). The size of  checkpoint file increases along with the size of the data, however once the file gets expanded, its size will not decrease even if data such as containers or rows are deleted. In this case, GridDB reuses the free space instead. Checkpoint files can be split so as to be stored in multiple disks.

  - Transaction log file
    
    Transaction data that are written to the database in memory is  perpetuated to the transaction log file by writing the data sequentially in a log format.

  - Definition file
    
    Definition file includes two types of parameter files: gs_cluster.json, hereinafter referred to as a cluster definition  file, used when composing a cluster; gs_node.json, hereinafter referred to as a node definition file, used to set the operations and resources of the node in a cluster. It also includes a user definition file for GridDB administrator users.

  - Event log file
    
    The event log of the GridDB server is saved in this file, including messages such as errors, warnings and so on.

  - Backup file
    
    Backup data in the data file of GridDB is saved.

![Database file](img/arc_DatabaseFile.png)


The placement of these resources is defined in GridDB home (path specified in environmental variable GS_HOME). In the initial installation state, the /var/lib/gridstore directory is GridDB home, and the initial data of each resource is placed under this directory.

The directories are placed initially as follows.

``` 
/var/lib/gridstore/        # GridDB home directory path
     admin/                # gs_admin home directory
     backup/               # Backup directory
     conf/                 # Definition files directory
          gs_cluster.json  # Cluster definition file
          gs_node.json     # Node definition file
          password         # User definition file
     data/                 # Database directory
     log/                  # Log directory
```

The location of GridDB home can be changed by setting the .bash_profile file of the OS user gsadm. If you change the location, please also move resources in the above directory accordingly.

The .bash_profile file contains two environment variables, GS_HOME and GS_LOG.

``` 
vi .bash_profile

# GridStore specific environment variables
GS_LOG=/var/lib/gridstore/log
export GS_LOG
GS_HOME=/var/lib/gridstore　　　　　　　　　　// GridDB home directory path
export GS_HOME
```

The database directory, backup directory and server event log directory can be changed by changing the settings of the node definition file as well.

See [Parameters](#parameters) for the contents that can be set in the cluster definition file and node definition file.


## Data access function

To access GridDB data, there is a need to develop an application using NoSQL I/F or NewSQL I/F. Data can be accessed simply by connecting to the cluster database of GridDB without having to take into account position information on where the container or table is located in the cluster database. The application system does not need to consider which node constituting the cluster the container is placed in.

In the GridDB API, when connecting to a cluster database initially, placement hint information of the container is retained (cached) on the client end together with the node information (partition).

Communication overheads are kept to a minimum as the node maintaining the container is connected and processed directly without having to access the cluster to search for nodes that have been placed every time the container used by the application is switched.

Although the container placement changes dynamically due to the rebalancing process in GridDB, the position of the container is transmitted as the client cache is updated regularly. For example, even when there is a node mishit during access from a client due to a failure or a discrepancy between the regular update timing and re-balancing timing, relocated information is automatically acquired to continue with the process.

<a id="tql_and_sql"></a>
### TQL and SQL

TQL and SQL-92 compliant SQL are supported as database access languages.

  - What is TQL?
    
    A simplified SQL prepared for GridDB SE. The support range is limited to functions such as search, aggregation, etc., using a container as a unit. TQL is employed by using the client API (Java, C language) of GridDB SE.
    
    The TQL is adequate for the search in the case of a small  container and a small number of hits. For that case, the response is faster than SQL. The number of hits can be suppressed by the LIMIT clause of TQL.
    
    For the search of a large amount of data, SQL is recommended.
    
    TQL is available for the containers and partitioned tables created by operations through the NewSQL interface. The followings are the limitations of TQL for the partitioned tables.
    
      - Filtering data by the WHERE clause is available. But aggregate functions, timeseries data selection or interpolation, min or max function and ORDER BY clause, etc. are not available.
    
      - It is not possible to apply the update lock.
    
    See "GridDB TQL Reference" ([GridDB_TQL_Reference](GridDB_TQL_Reference.md)) for details.

  - What is SQL?
    
    Standardization of the language specifications is carried out in ISO to support the interface for defining and performing data operations in conformance with SQL-92 in GridDB. SQL can be used in NewSQL I/F.
    
    SQL is also available for the containers created by operations through the NoSQL interface.
    
    See "GridDB SQL reference" ([GridDB_SQL_Reference](GridDB_SQL_Reference.md))     for details.

<a id="batch_functions"></a>
### Batch-processing function to multiple containers

An interface to quickly process event information that occurs occasionally is available in NoSQL I/F.

When a large volume of events is sent to the database server every time an event occurs, the load on the network increases and system throughput does not increase. Significant impact will appear especially when the communication line bandwidth is narrow. Multi-processing is available in NoSQL I/F to process multiple row registrations for multiple containers and multiple inquiries (TQL) to multiple containers with a single request. The overall throughput of the system rises as the database server is not accessed frequently.

An example is given below.

- Multi-put
    
    - A container is prepared for each sensor name as a process to register event information from multiple sensors in the database. The sensor name and row array of the timeseries event of the sensor are created and a list (map) summarizing the data for multiple sensors is created. This list data is registered in the GridDB database each time the API is invoked.
    
    - Multi-put API optimizes the communication process by combining requests of data registration into multiple containers to a node in GridDB, which is formed by multiple clusters. In addition, multi-registrations are processed quickly without performing MVCC when executing a transaction.
    
    - In a multi-put processing, transactions are committed automatically. Data is confirmed on a single case basis.


![Multi-put](img/func_multiput.png)



- Multi-query (fetchAll)
    
    - Instead of executing multiple queries to a container, these can be executed in a single query by aggregating event information of the sensor. For example, this is most suitable for acquiring aggregate results such as the daily maximum, minimum and average values of data acquired from a sensor, or data of a row set having the maximum or minimum value, or data of a row set meeting the specified condition.


![fetchAll](img/func_multiquery.png)


- Multi-get
    
    - Instead of executing multiple queries to a sensor, these can be executed in a single query by consolidating event information of the sensor. For example, this is most suitable for acquiring aggregate results such as the daily maximum, minimum and average values of data acquired from a sensor, or data of a row set having the maximum or minimum value, or data of a row set meeting the specified condition.
    
    - In a RowKeyPredicate object, the acquisition condition is set in either one of the 2 formats below.  
      - Specify the acquisition range
      - Specified individual value


![multi-get](img/func_multiget.png)


<a id="index_function"></a>
## Index function

A condition-based search can be processed quickly by creating an index for the columns of a container (table).

There are 3 types of index - hash index (HASH), tree index (TREE) and space index (SPATIAL). A hash index is used in an equivalent-value search when searching with a query in a container. Besides equivalent-value search, a tree index is used in comparisons including the range (bigger/same, smaller/same etc.).

The index that can be set differs depending on the container (table) type and column data type.

- HASH INDEX
    - An equivalent value search can be conducted quickly but this is not suitable for searches that read the rows sequentially.
    - Columns of the following data type can be set in a  collection. Cannot be set in a timeseries container, a table, and a timeseries table.
        - STRING
        - BOOL
        - BYTE
        - SHORT
        - INTEGER
        - LONG
        - FLOAT
        - DOUBLE
        - TIMESTAMP
- TREE INDEX
    - Besides equivalent-value search, a tree index is used in comparisons including the range (bigger/same, smaller/same  etc.).
    - This can be set for columns of the following data type in any type of container (table), except for columns corresponding to a rowkey in a timeseries container (timeseries table).
        - STRING
        - BOOL
        - BYTE
        - SHORT
        - INTEGER
        - LONG
        - FLOAT
        - DOUBLE
        - TIMESTAMP
    - Only a tree index allows an index with multiple columns, which is called a composite index. A composite index can be set up to 16 columns, where the same column cannot be specified more than once.
- SPATIAL INDEX
    - Can be set for only GEOMETRY columns in a collection. This is specified when conducting a spatial search at a high speed.

Although there are no restrictions on the no. of indices that can be created in a container, creation of an index needs to be carefully designed. An index is updated when the rows of a configured container are inserted, updated or deleted. Therefore, when multiple indices are created in a column of a row that is updated frequently, this will affect the performance in insertion, update or deletion operations.

An index is created in a column as shown below.
- A column that is frequently searched and sorted
- A column that is frequently used in the condition of the WHERE section of TQL
- High cardinality column (containing few duplicated values)

[Note]
- Only a tree index can be set to the column of a table (time series table).

<a id="ts_data_functions"></a>
## Function specific to time series data

To manage data frequently produced from sensors, data is placed in accordance with the data placement algorithm (TDPA: Time Series Data Placement Algorithm), which allows the best use of the memory. In a timeseries container (timeseries table), memory is allocated while classifying internal data by its periodicity. When hint information is given in an affinity function, the placement efficiency rises further. Expired data in a timeseries container is released at almost zero cost while being expelled to a disk where necessary.

A timeseries container (timeseries table) has a TIMESTAMP ROWKEY (PRIMARY KEY).

### Compression function

In timeseries container (timeseries table), data can be compressed and held. Data compression can improve memory usage efficiency. Compression options can be specified when creating a timeseries container (timeseries table).

However, the following row operations cannot be performed on a timeseries container (timeseries table) for which compression options are specified.

- Updating a specified row.
- Deleting a specified row.
- Inserting a new row when there is a row at a later time than the specified time.

The following compression types are supported:
- HI: thinning out method with error value
- NO: no compression.
- SS: thinning out method without error value

The explanation of each option is as follows.

#### Thinning out method with error value (HI).

When the previous and the following registered data lies in the same slope, the current data, which is represented by a row is omitted. The condition of the slope can be specified by the user.

The row data is omitted only when the specified column satisfies the condition and the values of the other columns are the same as the previous data. The condition is specified by the error width (Width).


![Compression of timeseries container (timeseries table)](img/func_TimeseriseCompression.png)


Compression can be enabled to the following data types:
- LONG
- INTEGER
- SHORT
- BYTE
- FLOAT
- DOUBLE

Since lossy compression is used, data omitted by the compression cannot be restored to its original value.

Omitted data will be restored without error value at the process of interpolate and sample processing.

#### Thinning out method without error value (SS)

With SS type, the row with the same data as the row registered just before and immediately after will be omitted. Omitted data will be restored without error value at the process of interpolate and sample processing.


### Operation function of TQL

#### Aggregate operations

In a timeseries container (timeseries table), the calculation is performed with the data weighted at the time interval of the sampled data. In other words, if the time interval is long, the calculation is carried out assuming the value is continued for an extended time.

The functions of the aggregate operation are as follows:

- TIME_AVG
    
    - TIME_AVG Returns the average weighted by a time-type key of values in the specified column.
    - The weighted average is calculated by dividing the sum of products of sample values and their respective weighted values by the sum of weighted values. The method for calculating a weighted value is as shown above.
    - The details of the calculation method are shown in the figure:


![Aggregation of weighted values (TIME_AVG)](img/func_TIME_AVG.png)


#### Selection/interpolation operation

Time data may deviate slightly from the expected time due to the timing of the collection and the contents of the data to be collected. Therefore when conducting a search using time data as a key, a function that allows data around the specified time to be acquired is also required.

The functions for searching the timeseries container (timeseries table) and acquiring the specified row are as follows:

- TIME_NEXT(*, timestamp)
    
  Selects a time-series row whose timestamp is identical with or just after the specified timestamp.

- TIME_NEXT_ONLY(*, timestamp)
    
  Select a time-series row whose timestamp is just after the specified timestamp.
- TIME_PREV(*, timestamp)
    
  Selects a time-series row whose timestamp is identical with or just before the specified timestamp.
- TIME_PREV_ONLY(*, timestamp)
    
  Selects a time-series row whose timestamp is just before the specified timestamp.

In addition, the functions for interpolating the values of the columns are as follows:

- TIME_INTERPOLATED(column, timestamp)
    
    Returns a specified column value of the time-series row whose timestamp is identical with the specified timestamp, or a value obtained by linearly interpolating specified column values of adjacent rows whose timestamps are just before and after the specified timestamp, respectively.

- TIME_SAMPLING(*|column, timestamp_start, timestamp_end, interval, DAY|HOUR|MINUTE|SECOND|MILLISECOND)
    
    Takes a sampling of Rows in a specific range from a given start time to a given end time.
    
    Each sampling time point is defined by adding a sampling interval multiplied by a non-negative integer to the start time, excluding the time points later than the end time.
    
    If there is a Row whose timestamp is identical with each  sampling time point, the values of the Row are used. Otherwise, interpolated values are used.

### Expiry release function

An expiry release is a function to delete expired row data from GridDB physically. The data becomes unavailable by removing from a target for a search or a delete before deleting. Deleting old unused data results to keep database size results in prevention of performance degradation caused by bloat of database size.


![Expiry release settings](img/func_expiration.png)


The retention period is set in container units. The row which is outside the retention period is called "expired data." The APIs become unable to operate expired data because it becomes unavailable. Therefore, applications can not access the row. Expired data will be the target for being deleted physically from GridDB after a certain period. The target is called "cold data." It is possible to delete it automatically from GridDB at the time and after saving to a external file.

#### Expiry release types

There are two setting types in the retention period. Use "row expiry release" for a time series container and use "partition expiry release" for a partitioned table.

- Row expiry release
    
  - It can be set for a time series container.
  - Setting items consist of a retention period, a retention period unit, and a division count.
  - The retention period unit can be set in day/hour/minute/sec/millisecond units. The year unit and month unit cannot be specified.
  - The expiration date of rows is calculated by adding row key stored date and time (retention period start date) to the retention period. It is calculated for every row.
  - The unit for becoming cold data is the rows in the period  (retention period/division count). For example, if the retention period is 720 days and the division count is 36, the rows in 20 (720/36) days become cold data. Physical data delete for the rows in 20 days is executed all at once.


![Row expiry release](img/func_expiration_row.png)



- Partition expiry release
    - It can be set for a table partitioned by interval and interval-hash. For a container, it can be set only if it has a partitioning key of a timestamp type.
    - Setting items consist of a retention period and a retention period unit.
    - The retention period unit can be set in day/hour/minute/sec/millisecond units. The year unit and month unit cannot be specified.
    - The expiration date of rows is calculated by adding the last date and time of the row stored period in the partition to the retention period. The rows stored in the same data partition have the same expiration date.
    - The unit becoming cold data is a data partition. Physical data delete is executed on a data partition basis. 
	(However, the row data is only deleted. Delete the data partition manually because it is not deleted. 
	See the "GridDB SQL Reference" ([GridDB_SQL_Reference)](GridDB_SQL_Reference.md) for the details of how to delete the data partition.


![Partition expiry release](img/func_expiration_partition.png)


Summary of the row expiry release and the partition expiry release


|                                | Row expiry release                                                                                | Partition expiry release                                                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Container type                 | Time series container                                                                             | Interval partitioning and interval-hash partitioning (For a collection, it can be set only if it has a partitioning key of a timestamp type.) |
| Setting items                  | Retention period, retention period unit, division count                                           | Retention period, retention period unit                                                                                                       |
| Expiration date                | Date and time adding the date and time when data is stored in the row key to the retention period | Date and time adding the last date and time of the row stored period in a partition to the retention period                                   |
| Unit for becoming expired data | Row                                                                                               | Data partition                                                                                                                                |
| Unit for becoming cold data    | Rows in the "retention period" divided by "division count"                                        | Data partition                                                                                                                                |

[Note]
  - Set for expiry release on the container creation. They cannot be changed after creation.
  - Both the row expiry release and the partition expiry release are set on the same container.
  - The current time used for judging expiration depends on the environment of each node of GridDB. Therefore, because of the network latency or time difference of the execution environments, you may not be able to access the rows in a GridDB node whose environment time is ahead of that of the client you use; on the contrary, you may be able to access the rows if the client you use is ahead of the time of GridDB. You had better set the period a larger value than you need to avoid unintentional loss of rows.
  - The expired rows are separated from the object of search and updating, being treated as not to exist in the GridDB. Operations to the expired row do not cause errors.  
      - For example, when you register a row with a timestamp of 31 days ago to the container with the expiration of 30 days, registration processing does not cause an error, while the row is not saved in the container.
  - When you set up expiry release, be sure to synchronize the environment time of all the nodes of a cluster. If the time is different among the nodes, the expired data may not be released at the same time among the nodes.
  - The period that expired data becomes cold data depends on the setting of the retention period in the expiry release.  
    - Retention period: Maximum period from expiration to cold data
	<br> ～3 days : about 1.2 hours
	<br> 3 days-12 days : about 5 hours
	<br> 12 days-48 days : about 19 hours
	<br> 48 days-192 days : about 3 days
	<br> 192 days-768 days : about 13 days
	<br> 768 days- : about 38 days

<a id="table_partitioning"></a>
## Table partitioning function

In order to improve the operation speed of applications connected to multiple nodes of the GridDB cluster, it is important to arrange the data to be processed in memory as much as possible. For huge table with a large number of rows, by distributing rows of the table to multiple nodes, processors and memory of multiple nodes can be effectively used. Distributed rows are stored in the internal containers called "data partition". The allocation of each row to the data partition is determined by a "partitioning key" column specified at the time of the table creation.

GridDB supports hash partitioning, interval partitioning and interval-hash partitioning as table partitioning methods.

Creating, Deleting tables and Data registration, update and search can be performed through the NewSQL interface. (There are some restrictions. See [TQL and SQL](#tql_and_sql) for the details.)

  - Data registration
    
    When data is registered into a table, the data is stored in the appropriate data partition according to the partitioning key value and the partitioning method. It is not possible to specify a data partition to be stored.

  - Index
    
    When creating an index on a table, a local index for each data partition is created. It is not possible to create a global index for the whole table.

  - Trigger
    
    Trigger can not be created to the partitioned table.

  - Data operation
    
    An error occurs for updating the partitioning key value of the primary key column. If updating is needed, delete and reregister the data.
    
    Updating the partitioning key value of the not primary key column can be executed through NoSQL I/F.

  - Functions of timeseries tables
    
    The expiry release function can be used for partitioned timeseries tables. The compression function cannot be used for the tables.

  - Notes
    
    From V4.3, if a column other than the primary key is specified as the partitioning key, an error will occur by default. To avoid this error, set false to /sql/partitioningRowkeyConstraint in the cluster definition file (gs_cluster.json).
    
    When specifying the column as a partitioning key other than the primary key, the primary key constraint is ensured in each data partition, but it is not ensured in the whole table. So, the same value may be registered in multiple rows of a table.


![Table partitioning](img/func_partitioning.png)

### Benefits of table partitioning

Dividing a large amount of data through a table partitioning is effective for efficient use of memory and for performance improvement in data search which can select the target data.

  - efficient use of memory
    
    In data registration and search, data partitions required for the processing are loaded into memory. Other data partitions, not target to the processing, are not loaded. So when the data to be processed is locally concentrated on some data partitions, the amount of loading data is reduced. The frequency of swap-in and swap-out is decreased and the performance is upgraded.

  - selecting target data in data search
    
    In data search, only data partitions matching the search condition are selected as the target data. Unnecessary data partitions are not accessed. This function is called "pruning". Because the amount of accessed data reduces, the search performance is upgraded. Search conditions which can enable the pruning are different depending on the type of the partitioning.

The followings describe the behaviors on the above items for both cases in not using the table partitioning and in using the table partition.

When a large amount of data is stored in single table which is not partitioned, all the required data might not be able to be placed on main memory and the performance might be degraded by frequent swap-in and swap-out between database files and memory. Particularly the degradation is significant when the amount of data is much larger than the memory size of a GridDB node. And data accesses to that table concentrate on single node and the parallelism of database processing decreases.


![When not using table partitioning](img/func_partitioning2.png)


By using a table partitioning, the large amount of data is divided into data partitions and those partitions are distributed on multiple nodes.

In data registration and search, only necessary data partitions for the processing can be loaded into memory. Data partitions not target to the processing are not loaded. Therefore, in many cases, data size required by the processing is smaller than for a not partitioned large table and the frequency of swap-in and swap-out decreases. By dividing data into data partitions equally, CPU and memory resource on each node can be used more effectively.

In addition data partitions are distributed on nodes, so parallel data access becomes possible.


![When using table partitioning](img/func_partitioning3.png)


### Hash partitioning

The rows are evenly distributed in the data partitions based on the hash value.

Also, when using an application system that performs data registration frequently, the access will concentrate at the end of the table and may lead to a bottleneck. A hash function that returns an integer from 1 to N is defined by specifying the partition key column and division number N, and division is performed based on the returned value.

![Hash partitioning](img/func_partitioning_hash.png)


  - Data partitioning
    
    By specifying the partitioning key and the division count M, a hash function that returns 1-M is defined, and data partitioning is performed by the hash value. The maximum hash value is 1024.

  - Partitioning key
    
    There is no limitation for the column type of a partitioning key.

  - Creation of data partitions
    
    Specified number of data partitions are created automatically at the time of the table creation. It is not possible to change the number of data partitions. The table re-creation is needed for changing the number.

  - Deletion of a table
    
    It is not possible to delete only a data partition.
    
    By deleting a hash partitioned table, all data partitions that belong to it are also deleted

  - Pruning
    
    In key match search on hash partitioning, by pruning, the search accesses only data partitions which match the condition. So the hash partitioning is effective for performance improvement and memory usage reduction in that case.

### Interval partitioning

In the interval partitioning, the rows in a table are divided by the specified interval value and is stored in data partitions. The range of each data partition (from the lower limit value to the upper limit value) is automatically determined by the interval value.

The data in the same range are stored in the same data partition, so for the continuous data or for the range search, the operations can be performed on fewer memory.


![Interval partitioning](img/func_partitioning_interval.png)


  - Data partitioning
    
    Data partitioning is performed by the interval value. The possible interval values are different depending on the partitioning key type.
      - BYTE type : from 1 to 2<sup>7</sup>-1
      - SHORT type : from 1 to 2<sup>15</sup>-1
      - INTEGER type : from 1 to 2<sup>31</sup>-1
      - LONG type : from 1000 to 2<sup>63</sup>-1
      - TIMESTAMP: more than 1
    
    When the partitioning key type is TIMESTAMP, it is necessary to specify the interval unit as 'DAY'.

  - Partitioning key
    
    Data types that can be specified as a partitioning key are BYTE, SHORT, INTEGER, LONG and TIMESTAMP. The partitioning key is a column that needs to have "NOT NULL" constraint.

  - Creation of data partitions
    
    Data partitions are not created at the time of creating the table. When there is no data partition for the registered row, a new data partition is automatically created.
    
    The upper limit of the number of the data partitions is 10000. When the number of the data partitions reaches the limit, the data registration that needs to create a new data partition causes an error. For that case, delete unnecessary data partitions and reregister the data.
    
    It is desired to specify the interval value by considering the range of the whole data and the upper limit, 10000, for the number of data partitions. If the interval value is too small to the range of the whole data and too many data partitions are created, the maintenance of deleting unnecessary data partitions is required frequently.

  - Deletion of a table
    
    Each data partition can be deleted. The data partition that has been deleted cannot be recreated. 
	All registration operations to the deleted data partition cause an error. 
	Before deleting the data partition, check its data range by a metatable. 
	See the "[GridDB SQL Reference](GridDB_SQL_Reference)" (GridDB_SQL_Reference.md) for the details of the metatable schema.
    
    By deleting an interval partitioned table, all data partitions that belong to it are also deleted.
    
    If the row expiry release function is set, the data partition that becomes empty for the expiration is not deleted automatically. 
	All data partitions are processed for the data search on the whole table, so the search can be performed efficiently by deleting unnecessary data partitions beforehand.

  - Maintenance of data partitions
    
    In the case of reaching the upper limit, 10000, for the number of data partitions or existing unnecessary data partitions, the maintenance by deleting data partitions is needed.
    
      - How to check the number of data partitions
        
        It can be check by search the metatable that holds the data about data partitions. See "GridDB SQL  reference" ([GridDB_SQL_Reference](GridDB_SQL_Reference.md)) for details.
    
      - How to delete data partitions
        
        They can be deleted by specifying the lower limit value in the data partition. See "GridDB SQL reference" ([GridDB_SQL_Reference](GridDB_SQL_Reference.md)) for the details.


![Examples of interval partitioned table creation and deletion](img/func_partitioning_interval2.png)


  - Pruning
    
    By specifying a partitioning key as a search condition in the WHERE clause, the data partitions corresponding the specified key are only referred for the search, so that the processing speed and the memory usage are improved.

### Interval-hash partitioning

The interval-hash partitioning is a combination of the interval partitioning and the hash partitioning. First the rows are divided by the interval partitioning, and further each division is divided by hash partitioning. The number of data partitions is obtained by multiplying the interval division count and the hash division count together.


![Interval-hash partitioning] (img/func_partitioning_interval_hash.png)

The rows are distributed to multiple nodes appropriately through the hash partitioning on the result of the interval partitioning. On the other hand, the number of data partitions increases, so that the overhead of searching on the whole table also increases. Please judge to use the partitioning by considering its data distribution and search overhead.

The basic functions of the interval-hash partitioning are the same as the functions of interval partitioning and the hash partitioning. The items specific for the interval-hash partitioning are as follows.

  - Data partitioning
    
    The possible interval values of LONG are different from the interval partitioning.
      - BYTE type : from 1 to 2<sup>7</sup>-1
      - SHORT type : from 1 to 2<sup>15</sup>-1
      - INTEGER type : from 1 to 2<sup>31</sup>-1
      - LONG type : from 1000 * hash division count to 2<sup>63</sup>-1
      - TIMESTAMP: more than 1

  - Number of data partitions
    
    Including partitions divided by hash, the upper limit of number of data partitions is 10000. The behavior and requiring maintenance when the limit has been reached are same as interval partitioning.

  - Deletion of a table
    
    A group of data partitions which have the same range can be deleted. It is not possible to delete only a data partition divided by the hash partitioning.

### Selection criteria of table partitioning type

Hash, interval and interval-hash are supported as a type of table partitioning by GridDB.

A column which is used in conditions of search or data access must be specified as a partitioning key for dividing the table. If a width of range that divides data equally can be determined for values of the partitioning key, interval or interval-hash is suitable. Otherwise hash should be selected.


![Data range](img/func_partitioning4.png)


  - Interval partitioning, interval-hash partitioning
    
    If an interval, a width of range to divide data equally, can be determined beforehand, interval partitioning is suitable. In the query processing on interval partitioning, by partitioning pruning, the result is acquired from only the data partitions matching the search condition, so the performance is improved. Besides the query processing, when registering successive data in a specific range, performance is improved.
    
	
    ![Interval partitioning](img/func_partitioning_interval3.png)
    
	
    Therefore, when using interval partitioning, by selecting an appropriate interval value based on frequently registered or searched data range in application programs, required memory size is reduced. For example, in the case of a system in which time-series data such as sensor data is frequently searched for the latest data, if the search target range is the division width value of interval partitioning, the search is performed in the memory of the data partition size to be processed and the performance dose not deteriorates.
    
	
    ![Examples of data registration and search on interval     partitioning](img/func_partitioning_interval4.png)
    
    Further by using interval-hash partitioning, data in each interval is distributed to multiple nodes equally, so accesses to the same data partition can also be performed in parallel.
    
	
    ![Interval-hash partitioning] (img/func_partitioning_interval_hash2.png)


  - Hash partitioning
    
    When the characteristics of data to be stored is not clear or finding the interval value, which can divide the data equally, is difficult, hash partitioning should be selected. By specifying a column holding unique values as a partitioning key, uniform partitioning for a large amount of data is performed automatically.
    
	
    ![Hash partitioning](img/func_partitioning_hash2.png)
    
	
    When using hash partitioning, the parallel access to the entire table and the partitioning pruning which is enabled only for exact match search can be performed, so the system performance can be improved. But, to obtain high performance, each node is required to have enough memory that can store the entire data partition of the node.
    
	
    ![Examples of data registration and search on hash partitioning](img/func_partitioning_hash3.png)


## Transaction function

GridDB supports transaction processing on a container basis and ACID characteristics which are generally known as transaction characteristics. The supporting functions in a transaction process are explained in detail below.

### Starting and ending a transaction

When a row search or update etc. is carried out on a container, a new transaction is started and this transaction ends when the update results of the data are committed or aborted.

[Note]
  - A commit is a process to confirm transaction information under processing to perpetuate the data.
      - In GridDB, updated data of a transaction is stored as a transaction log by a commit process, and the lock that had been maintained will be released.
  - An abort is a process to rollback (delete) all transaction data under processing.
      - In GridDB, all data under processing are discarded and retained locks will also be released.

The initial action of a transaction is set in autocommit.

In autocommit, a new transaction is started every time a container is updated (data addition, deletion or revision) by the application, and this is automatically committed at the end of the operation. A transaction can be committed or aborted at the requested timing by the application by turning off autocommit.

A transaction recycle may terminate in an error due to a timeout in addition to being completed through a commit or abort. If a transaction terminates in an error due to a timeout, the transaction is aborted. The transaction timeout is the elapsed time from the start of the transaction. Although the initial value of the transaction timeout time is set in the definition file (gs_node.json), it can also be specified as a parameter when connecting to GridDB on an application basis.

### Transaction consistency level

There are 2 types of transaction consistency levels, immediate consistency and eventual consistency. This can also be specified as a parameter when connecting to GridDB for each application. The default setting is immediate consistency.

  - immediate consistency
    
      - Container update results from other clients are reflected  immediately at the end of the transaction concerned. As a result, the latest details can be referenced all the time.

  - eventual consistency
    
      - Container update results from other clients may not be reflected immediately at the end of the transaction concerned. As a result, there is a possibility that old details may be referred to.

Immediate consistency is valid in update operations and read  operations. Eventual consistency is valid in read operations only. For applications which do not require the latest results to be read all the time, the reading performance improves when eventual consistency is specified.

### Transaction isolation level

Conformity of the database contents need to be maintained all the time. When executing multiple transaction simultaneously, the following events will generally surface as issues.

  - Dirty read
    
    An event which involves uncommitted data written by a dirty read transaction being read by another transaction.

  - Non-repeatable read
    
    An event which involves data read previously by a non-repeatable read transaction becoming unreadable. Even if you try to read the data read previously by a transaction again, the previous data can no longer be read as the data has already been updated and committed by another transaction (the new data after the update will be read instead).

  - Phantom read
    
    An event in which the inquiry results obtained previously by a phantom read transaction can no longer be acquired. Even if you try to execute an inquiry executed previously in a transaction again in the same condition, the previous results can no longer be acquired as the data satisfying the inquiry condition has already been changed, added and committed by another transaction (new data after the update will be acquired instead).

In GridDB, "READ_COMMITTED" is supported as a transaction isolation level. In READ_COMMITTED, the latest data confirmed data will always be read.

When executing a transaction, this needs to be taken into consideration so that the results are not affected by other transactions. The isolation level is an indicator from 1 to 4 that shows how isolated the executed transaction is from other transactions (the extent that consistency can be maintained).

The 4 isolation levels and the corresponding possibility of an event raised as an issue occurring during simultaneous execution are as follows.

| Isolation level   | Dirty read                | Non-repeatable read       | Phantom read              |
| ----------------- | ------------------------- | ------------------------- | ------------------------- |
| READ_UNCOMMITTED | Possibility of occurrence | Possibility of occurrence | Possibility of occurrence |
| READ_COMMITTED   | Safe                      | Possibility of occurrence | Possibility of occurrence |
| REPEATABLE_READ  | Safe                      | Safe                      | Possibility of occurrence |
| SERIALIZABLE      | Safe                      | Safe                      | Safe                      |

In READ_COMMITED, if data read previously is read again, data that is different from the previous data may be acquired, and if an inquiry is executed again, different results may be acquired even if you execute the inquiry with the same search condition. This is because the data has already been updated and committed by another transaction after the previous read.

In GridDB, data that is being updated by MVCC is isolated.

### MVCC

In order to realize READ_COMMITTED, GridDB has adopted "MVCC (Multi-Version Concurrency Control)".

MVCC is a processing method that refers to the data prior to being updated instead of the latest data that is being updated by another transaction when a transaction sends an inquiry to the database. System throughput improves as the transaction can be executed concurrently by referring to the data prior to the update.

When the transaction process under execution is committed, other transactions can also refer to the latest data.


![MVCC](img/func_MVCC.png)


### Lock

There is a data lock mechanism to maintain the consistency when there are competing container update requests from multiple transactions.

The lock granularity differs depending on the type of container. In addition, the lock range changes depending on the type of operation in the database.

#### Lock granularity

The lock granularity for each container type is as follows.

  - Collection: Lock by ROW unit.
  - Timeseries container: Locked by ROW collection
      - In a ROW collection, multiple rows are placed in a timeseries container by dividing a block into several data processing units. This data processing unit is known as a row set. It is a data management unit to process a large volume of timeseries containers at a high speed even though the data granularity is coarser than the lock granularity in a collection.

These lock granularity were determined based on the use-case analysis of each container type.

  - Collection data may include cases in which an existing ROW data is updated as it manages data just like a RDB table.
  - A timeseries container is a data structure to hold data that is being generated with each passing moment and rarely includes cases in which the data is updated at a specific time.

#### Lock range by database operations

Container operations are not limited to just data registration and deletion but also include schema changes accompanying a change in data structure, index creation to improve speed of access, and other operations. The lock range depends on either operations on the entire container or operations on specific rows in a container.

  - Lock the entire container   
      - Index operations (createIndex/dropIndex)
      - Deleting container
      - Schema change
  - Lock in accordance with the lock granularity   
      - put/update/remove
      - get(forUpdate)
    
    In a data operation on a row, a lock following the lock granularity is ensured.

If there is competition in securing the lock, the subsequent transaction will be put in standby for securing the lock until the earlier transaction has been completed by a commit or rollback process and the lock is released.

A standby for securing a lock can also be cancelled by a timeout besides completing the execution of the transaction.

### Data perpetuation

Data registered or updated in a container or table is perpetuated in the disk or SSD, and protected from data loss when a node failure occurs. There are 2 types of transaction log process, one to synchronize data in a data update and write the updated data sequentially in a transaction log file, and the other is a checkpoint process to store updated data in the memory regularly in the database file on a block basis.

To write to a transaction log, either one of the following settings can be made in the node definition file.

  - 0: SYNC
  - An integer value of 1 or higher: DELAYED_SYNC

In the "SYNC" mode, log writing is carried out synchronously every time an update transaction is committed or aborted. In the "DELAYED_SYNC" mode, log writing during an update is carried out at a specified delay of several seconds regardless of the update timing. Default value is "1 (DELAYED_SYNC 1 sec)".

When "SYNC" is specified, although the possibility of losing the latest update details when a node failure occurs is lower, the performance is affected in systems that are updated frequently.

On the other hand, if "DELAYED_SYNC" is specified, although the update performance improves, any update details that have not been written in the disk when a node failure occurs will be lost.

If there are 2 or more replicas in a raster configuration, the possibility of losing the latest update details when a node failure occurs is lower even if the mode is set to "DELAYED_SYNC" as the other nodes contain replicas. Consider setting the mode to "DELAYED_SYNC" as well if the update frequency is high and performance is required.

In a checkpoint, the update block is updated in the database file. A checkpoint process operates at the cycle set on a node basis. A checkpoint cycle is set by the parameters in the node definition file. Initial value is 60 sec (1 minute).

By raising the checkpoint execution cycle figure, data perpetuation can be set to be carried out in a time band when there is relatively more time to do so e.g. by perpetuating data to a disk at night and so on. On the other hand, when the cycle is lengthened, the disadvantage is that the number of transaction log files that have to be rolled forward when a node is restarted outside the system process increases, thereby increasing the recovery time.

The data updated at a checkpoint is collected and maintained in a memory different from the block in which the data was wrote at the checkpoint. Set up concurrent execution of checkpoints for faster checkpoint processing. When the concurrent execution is set up, up to as many as the number of concurrent execution of a transaction, checkpoints are processed concurrently.


![Checkpoint](img/func_checkpnt.png)


### Timeout process

NoSQL I/F and a NewSQL I/F have different setting items for timeout processing.

#### NoSQL I/F timeout process

In the NoSQL I/F, 2 types of timeout could be notified to the application developer, Transaction timeout and Failover timeout. The former is related to the processing time limit of a transaction, and the latter is related to the retry time of a recovery process when a failure occurs.

  - TransactionTimeout
    
    The timer is started when access to the container subject to the process begins, and a timeout occurs when the specified time is exceeded.
    
    Transaction timeout is configured to delete lock, and memory from a long-duration update lock (application searches for data in the update mode, and does not delete when the lock is maintained) or a transaction that maintains a large amount of results (application does not delete the data when the lock is maintained). Application process is aborted when transaction timeout is triggered.
    
    A transaction timeout time can be specified in the application with a parameter during cluster connection. The upper limit of this can be specified in the node definition file. The default value of upper limit is 300 seconds. To monitor transactions that take a long time to process, enable the timeout setting and set a maximum time in accordance with the system requirements.

  - FailoverTimeout
    
    Timeout time during an error retry when a client connected to a node constituting a cluster which failed connects to a replacement node. If a new connection point is discovered in the retry process, the client application will not be notified of the error. Failover timeout is also used in timeout during initial connection.
    
    A failover timeout time can be specified in the application by a parameter during cluster connection. Set the timeout time to meet the system requirements.

Both the transaction timeout and failover timeout can be set when connecting to a cluster using a GridDB object in the Java API or C API. See "GridDB Java API reference"
([GridDB_Java_API_Reference.html)](GridDB_Java_API_Reference.html)and "GridDB C API reference" ([GridDB_C_API_Reference.html](GridDB_C_API_Reference.html)) for details.

　

#### NoSQL I/F timeout process

There are 3 types of timeout as follows:

  - Login (connection) timeout
    
    Timeout for initial connection to the cluster. The default value is 300 seconds (5 minutes) and can be changed using DriverManager of API .

  - Network timeout
    
    Timeout in response between client and cluster. The timeout time is 300 seconds (5 minutes) and can not be changed in the current GridDB version.
    
    If the server does not respond for 15 seconds during communication from the client, it will retry, and if there is no response for 5 minutes it will timeout. There is no timeout during long-term query processing.

  - Query timeout
    
    Timeout time can be specified for each query to be executed. The default value for the timeout time is not set, allowing long-term query processing. In order to monitor the long-term query, set the timeout time according to the requirements of the system. The setting can be specified by Statement of the API.


## Replication function

Data replicas are created on a partition basis in accordance with the number of replications set by the user among multiple nodes constituting a cluster.

A process can be continued non-stop even when a node failure occurs by maintaining replicas of the data among scattered nodes. In the client API, when a node failure is detected, the client automatically switches access to another node where the replica is maintained.

The default number of replication is 2, allowing data to be replicated twice when operating in a cluster configuration with multiple nodes.

When there is an update in a container, the owner node (the node having the master replica) among the replicated partitions is updated.

There are 2 ways of subsequently reflecting the updated details from the owner node in the backup node.

  - Asynchronous replication
    
    Replication is carried out without synchronizing with the timing of the asynchronous replication update process. Update performance is better for quasi-synchronous replication but the availability is worse.

  - Quasi-synchronous replication
    
    Although replication is carried out synchronously at the quasi-synchronous replication update process timing, no appointment is made at the end of the replication. Availability is excellent but performance is inferior.

If performance is more important than availability, set the mode to asynchronous replication and if availability is more important, set it to quasi-synchronous replication.

[Note]
  - The number of replications is set in the cluster definition file (gs_cluster.json) /cluster/replicationNum. Synchronous settings of the replication are set in the cluster definition file (gs_cluster.json) /transaction/replicationMode.



## Affinity function

An affinity is a function to connect related data. There are 2 types of affinity function in GridDB, data affinity and node affinity.

### Data affinity function

A data affinity is a function to raise the memory hit rate by arranging highly correlated data in the same block and localizing data access. By raising the memory hit ratio, the no. of memory mishits during data access can be reduced and the throughput can be improved. By using data affinity, even machines with a small memory can be operated effectively.

The data affinity settings provide hint information as container properties when creating a container (table). The characters that can be specified for the hint information are restricted by naming rules that are similar to those for the container (table) name. Data with the same hint information is placed in the same block as much as possible.

Data affinity hints are set separately by the data update frequency and reference frequency. For example, consider the data structure when system data is registered, referenced or updated by the following operating method in a system that samples and refers to the data on a daily, monthly or annual basis in a monitoring system.

1.  Data in minutes is sent from the monitoring device and saved in the container created on a monitoring device basis.
2.  Since data reports are created daily, one day's worth of data is aggregated from the data in minutes and saved in the daily container 
3.  Since data reports are created monthly, daily container (table) data is aggregated and saved in the monthly container
4.  Since data reports are created annually, monthly container (table) data is aggregated and saved in the annual container
5.  The current space used (in minutes and days) is constantly updated and displayed in the display panel.

In GridDB, instead of occupying a block in a container unit, data close to the time is placed in the block. Therefore, refer to the daily container (table) in 2., perform monthly aggregation and use the aggregation time as a ROWKEY (PRIMARY KEY). The data in 3. and the data in minutes in 1. may be saved in the same block.

When performing yearly aggregation (No.4 above) of a large amount of data, the data which need constant monitoring (No.1) may be swapped out. This is caused by reading the data, which is stored in different blocks (No.4 above), into the memory that is not large enough for all the monitoring data.

In this case, by providing hints to the container (table) according to the container (table) access frequency using a data affinity e.g. on a minute, daily or monthly basis, etc., data with a low access frequency and data with a high access frequency is separated into different blocks when the data is placed.

In this way, data can be placed to suit the usage scene of the application by the data affinity function.


![Data Affinity](img/feature_data_afinity.png)

### Data affinity function

Node affinity is a function to reduce the network load when accessing data by arranging highly correlated containers and tables in the same node. Although there is no container JOIN operation In the TQL of a NoSQL product, a table JOIN operation can be described in the SQL of a SQL product. When joining a table, the network access load of a table placed in another node of the cluster can be reduced. In addition, since concurrent processing using multiple nodes is no longer possible, there is no effect on shortening the turnaround time. Nonetheless, throughput may still rise due to a reduction in the network load.


![Placement of container/table based on node affinity](img/func_Node_Affinity.png)



To use the node affinity function, hint information is given in the container (table) name when the container (table) is created. A container (table) with the same hint information is placed in the same partition. Specify the container name as shown below.

  - Container (table) name@node affinity hint information

The naming rules for node affinity hint information are the same as the naming rules for the container (table) name.



## Trigger function

A trigger function is an automatic notification function using Java Messaging Service (JMS) or REST, when an operation (add/update or delete) is carried out on the row data of a container. Event notifications can be received without the need to poll and monitor database updates in the application system.


![Action of a trigger function](img/func_trigger.png)


  - Notification method   
      - There are 2 ways of notifying the application system.
          - Java Messaging Service(JMS)
          - REST

  - When the operating target is a single node   
      - The following three operations are available: setting a trigger, unsetting the trigger, and acquiring the settings of the trigger.

  - Timing of notice   
      - Notify when a row is created, updated, or deleted.
      - Notify before a replication is completed. When not in automatic commitment mode, notify while un-committed.

  - Contents of notice   
      - Notify a container name and the type of operation: creating, updating, or deletion a row.
      - When a column is specified to be noticed, the value of the column which includes the operated row is also in the notice.

  - Processing when an error occurs   
      - When an error occurs at the time of a notice, error information is recorded in an event log. The notice is not sent again after recovering from the failure.

  - Others  
      - When more than one rows are created and/or updated, a notice is issued for each row. For Java API, this processing is equivalent to the call of Container#put (java.util.Collection) or GridStore#multiPut (Map).
      - When a schema is changed in a container with a trigger setting, the setting will be effective in the changed container. The column which is not in the changed schema will be automatically deleted from the column name to be noticed.
      - Both JMS and REST notice can be set to a container, but should be set under different trigger names.


[Note]
  - Caution about the number of triggers and updating performance
      - Updating performance decreases as the increase in the number of containers with an active trigger and the number of active triggers. Set only the minimum necessary triggers.
  - Caution about the processing performance of the destination server of the notice
      - When the throughput of the destination server is extremely lower than that of the update process of GridDB, trigger process may fail and an error message may be recorded in an event log. When you update frequently the container with a trigger, consider the performance of the destination server.



## Change the definition of a container (table)


It is possible to change the definition such as addition of columns after creating a container. Changeable operations and APIs are following.

| When the operating target is a single node | NoSQL API | JDBC |
| ------------------------------------------ | --------- | ---- |
| Add column(tail)                           | ✓         | ✓    |
| Add column(except for tail)                | ✓ (*1)   | ✗    |
| Delete column                              | ✓ (*1)   | ✗    |

  - (*1) If you add columns except to the tail or delete columns, the container is recreated internally. Therefore, it takes a long time to operate the container whose data amount is large.
  - It is not possible to operate except for the above (e.g. change of the container name or the column name).

### Add column

Add a new column to a container.

  - NoSQL API   
      - Add a column with GridStore#putContainer. 
      - Get container information "ContainerInfo" from an existing container. Execute putContainer after setting a new column to container information. 
	  See "GridDB Java API reference"     ([GridDB_Java_API_Reference.html)](GridDB_Java_API_Reference.html) for details.
    
      - [Example program]     
        ``` java
        // Get container information
        ContainerInfo conInfo = store.getContainerInfo("table1");
        List<ColumnInfo> newColumnList = new ArrayList<ColumnInfo>();
        for ( int i = 0; i < conInfo.getColumnCount(); i++ ){
            newColumnList.add(conInfo.getColumnInfo(i));
        }
        // Set a new column to the tail
        newColumnList.add(new ColumnInfo("NewColumn", GSType.INTEGER));
        conInfo.setColumnInfoList(newColumnList);
        
        // Add a column
        store.putCollection("table1", conInfo, true);
        ```

  - JDBC    
      - Add a column with ALTER TABLE syntax of SQL.
      - In the case of SQL only the operation of adding a column to the tail is available. See "GridDB SQL reference" ([GridDB_SQL_Reference](GridDB_SQL_Reference.md)) for details.

If you obtain existing rows after adding columns, the "empty value" defined in the data type of each column as a additional column value returns. 
See Container<K,R> of a "GridDB Java API reference" ([GridDB_Java_API_Reference.html](GridDB_Java_API_Reference.html)) for details about the empty value. 
(In V4.1, there is a limitation "Getting existing rows after addition of a column results in NULL return from columns without NOT NULL constraint.")


![Example of adding an column](img/add_column.png)


### Delete column

Delete a column. It is only operational with NoSQL APIs.

  - NoSQL API
      - Delete a column with GridStore#putContainer. Get container information "ContainerInfo" from an existing container at first. Then, execute putContainer after excluding column information of a deletion target. 
	  See "GridDB Java API reference" ([GridDB_Java_API_Reference.html)](GridDB_Java_API_Reference.html) for details.


## Database compression/release function


<a id="block_data_compression"></a>
### Block data compression

When GridDB writes in-memory data to the database file residing on the disk, a database with larger capacity independent to the memory size can be obtained. However, as the size increases, so does the cost of the storage. To reduce the cost, the database file (checkpoint file) can be effectively compressed using GridDB's block data compression. In this case, flash memory with a higher price per unit of capacity can be utilized much more efficiently than HDD.

**Compression method**

When exporting in-memory data to the database file (checkpoint file), compression is performed to each block of GridDB write unit. The vacant area of Linux's file space due to compression can be deallocated, thereby reducing disk usages.

**Supported environment**

Since block data compression uses the Linux function, it depends on the Linux kernel version and file system. Block data compression is supported in the following environment.
  - OS: RHEL / CentOS 7.2 and later
  - File system: XFS
  - File system block size: 4 KB

　If block data compression is enabled in other environments, the GridDB node will fail to start.

**Configuration method**

The compression function needs to be configured in every nodes.

  - Set the following string in the node definition file (gs_node.json) /dataStore/storeCompressionMode.
      - To disable compression functionality: NO_COMPRESSION (default) 
      - To enable compression functionality: COMPRESSION
  - The settings will be applied after GridDB node is restarted.
  - By restarting GridDB node, enable/disable operation of the compression function can be changed.

[Note]
  - Block data compression can only be applied to checkpoint file. Transaction log files, backup file, and GridDB's in-memory data are not subject to compression.
  - Due to block data compression, checkpoint file will become sparse file.
  - Even if the compression function is changed effectively, data already written to the checkpoint file cannot be compressed.

### Deallocation of unused data blocks

The deallocation of unused data blocks is the function that reduces the size (disk space) of database files by the Linux file block deallocation  processing on unused block areas of database files (checkpoint files).

Use this function in the following cases.

  - A large amount of data has been deleted
  - There is no plan to update data and it is necessary to keep the DB for a long term.
  - The disk becomes full when updating data and reducing the DB size is needed temporarily.

The processing for the deallocation of unused blocks, the support environment and the execution method are explained below.

**Processing for deallocation**

The unused blocks of database files (checkpoint files) are deallocated in a GridDB node at the time of starting the node. Those remain deallocated until data is updated on them.

**Supported environment**

The support environment is the same as the [block data compression](#block_data_compression).

**Execution method**

Specify the deallocation option, --releaseUnusedFileBlocks, of the gs_startnode command, in the time of starting GridDB nodes.

Check the size of unused blocks and allocated blocks by the following command.
  - Items shown by the gs_stat command
      - storeTotalUse
        
        The total size of used blocks in the checkpoint files (bytes)
    
      - checkpointFileAllocateSize
        
        The total size of allocated blocks in the checkpoint files (bytes)

It is desired to perform this function when the size of allocated and unused blocks is large (storeTotalUse << checkpointFileAllocateSize).

[Note]
  - This function is available only for the checkpoint files. It is not available for the transaction log files and backup files.
  - The checkpoint files become sparse files by performing this function.
  - The disk usage can be reduced by this function, but it is possible to be a disadvantage of the performance by the fragmentations of sparse files.
  - The start-up of GridDB with this function may take more time than the normal start-up.

---
[//]: # (<a id="label_admin"></a>)
# Admin function

## User management function

There are 2 types of GridDB user, an OS user which is created during installation and a GridDB user to perform operations/development in GridDB (hereinafter referred to a GridDB user).

### OS user

An OS user has the right to execute operating functions in GridDB and a gsadm user is created during GridDB installation. This OS user is hereinafter referred to gsadm.

All GridDB resources will become the property of gsadm. In addition, all operating commands in GridDB are executed by a gsadm.

Authentication is performed to check whether the user has the right to connect to the GridDB server and execute the operating commands. This authentication is performed by a GridDB user.

### GridDB user　

  - Administrator user and general user
    
    There are 2 types of GridDB user, an administrator user and a general user, which differ in terms of which functions can be used. Immediately after the installation of GridDB, 2 users, a system and an admin user, are registered as default administrator users.
    
    An administrator user is a user created to perform GridDB operations while general users are users used by the application system.
    
    For security reasons, administrator users and general users need to be used differently according to the usage purpose.

  - Creating a user
    
    An administrator user can register or delete a gsadm, and the information is saved in the password file of the definition file directory as a GridDB resource. As an administrator user is saved/managed in a local file of the OS, it has to be placed so that the settings are the same in all the nodes constituting the cluster. In addition, administrator users need to be set up prior to starting the GridDB server. After the GridDB server is started, administrative users are not valid even if they are registered.
    
    An administrator user can create a general user after starting cluster operations in GridDB. A general user cannot be registered before the start of cluster services. A general user can only be registered using NewSQL interface against a cluster as it is created after a cluster is composed in GridDB and maintained as management information in the GridDB database.
    
    Since information is not communicated automatically among clusters, an administrator user needs to make the same settings in all the nodes and perform operational management such as determining the master management node of the definition file and distributing information from the master management node to all the nodes that constitute the cluster.


![GridDB users](img/func_user.png)



  - Rules when creating a user
    
    There are naming rules to be adopted when creating a user name.   
      - Administrator user: Specify a user starting with "gs#". After "gs#", the name should be composed of only alphanumeric characters and the underscore mark. Since the name is not case-sensitive, gs#manager and gs#MANAGER cannot be registered at the same time.
    
      - General user: Specify using alphanumeric characters and the underscore mark. The container name should not start with a number. In addition, since the name is not case-sensitive, user and USER cannot be registered at the same time. System and admin users cannot be created as default administrator users.
    
      - Password: No restrictions on the characters that can be specified.
    
    A string of up to 64 characters can be specified for the user name and password.

### Usable function

The operations available for an administrator and a general user are as follows. Among the operations, commands which can be executed by a gsadm without using a GridDB user are marked with "✓✓".

| When the operating target is a single node | Operating details                              | Operating command and I/F used     | gsadm | Administrator user | General user                                                |
| ------------------------------------------ | ---------------------------------------------- | ------------------------ | ----- | ------------------ | ----------------------------------------------------------- |
| Node operations                            | start node                                     | gs_startnode     |       | ✓                  | ✗                                                           |
|                                            | stop node                                      | gs_stopnode      |       | ✓                  | ✗                                                           |
| Cluster operations                         | Building a cluster                             | gs_joincluster   |       | ✓                  | ✗                                                           |
|                                            | Detaching a node from a cluster                | gs_leavecluster  |       | ✓                  | ✗                                                           |
|                                            | Stopping a cluster                             | gs_stopcluster   |       | ✓                  | ✗                                                           |
| User management                            | Registering an administrator user              | gs_adduser              | ✓✓    | ✗                  | ✗                                                           |
|                                            | Deletion of administrator user                 | gs_deluser              | ✓✓    | ✗                  | ✗                                                           |
|                                            | Changing the password of an administrator user | gs_passwd               | ✓✓    | ✗                  | ✗                                                           |
|                                            | Creating a general user                        | NewSQL I/F                   |       | ✓                  | ✗                                                           |
|                                            | Deleting a general user                        | NewSQL I/F                   |       | ✓                  | ✗                                                           |
|                                            | Changing the password of a general user        | NewSQL I/F                   |       | ✓                  | ✓: Individual only                                          |
| Database management                        | Creating/deleting a database                   | NewSQL I/F                   |       | ✓                  | ✗                                                           |
|                                            | Assigning/cancelling a user in the database    | NewSQL I/F                   |       | ✓                  | ✗                                                           |
| Data operation                             | Creating/deleting a container or table         | NoSQL/NewSQL I/F                   |       | ✓                  | O : Only when update operation is possible in the user's DB |
|                                            | Registering data in a container or table       | NoSQL/NewSQL I/F                   |       | ✓                  | O : Only when update operation is possible in the user's DB |
|                                            | Searching for a container or table             | NoSQL/NewSQL I/F                   |       | ✓                  | ✓: Only in the DB of the individual                         |
|                                            | Creating index to a container or table         | NoSQL/NewSQL I/F                   |       | ✓                  | O : Only when update operation is possible in the user's DB |


### Database and users

Access to a cluster database in GridDB can be separated on a user basis. The separation unit is known as a database. The following is a cluster database in the initial state.

  - public
      - The database can be accessed by all administrator user and general users.
      - This database is used when connected without specifying the database at the connection point.

Multiple databases can be created in a cluster database. Creation of databases and assignment to users are carried out by an administrator user.

The rules for creating a database are as shown below.

  - The maximum no. of users and the maximum no. of databases that can be created in a cluster database is 128.
  - A string consisting of alphanumeric characters, the underscore mark, the hyphen mark, the dot mark, the slash mark and the equal mark can be specified for the database. The container name should not start with a number.
  - A string consisting of 64 characters can be specified for the database name.
  - Although the case sensitivity of the database name is maintained, a database which has the same name when it is not case-sensitive cannot be created. For example, both database and DATABASE cannot be registered.
  - Public and "information_schema" cannot be specified for default DB.

When assigning general users to a database, specify permissions as follows :
  - ALL
      - All operations to a container are allowed such as creating a container, adding a row, searching, and creating an index.
  - READ
      - Only search operations are allowed.

Only assigned general users and administrator users can access the database. Administrator user can access all databases. The following rules apply when assign a general user to a database.
  - Multiple general users can be assigned to one database.
  - When assigning general users to a database, only one type of permission can be granted.
  - When assigning multiple general users to one database, different permission can be granted for each user.
  - Multiple databases can be assigned to 1 user


![Database and users](img/func_database.png)

## Failure process function

In GridDB, recovery for a single point failure is not necessary as replicas of the data are maintained in each node constituting the cluster. The following action is carried out when a failure occurs in GridDB.


1.  When a failure occurs, the failure node is automatically isolated from the cluster.
2.  Failover is carried out in the backup node in place of the isolated failure node.
3.  Partitions are rearranged autonomously as the number of nodes decreases as a result of the failure (replicas are also arranged).

A node that has been recovered from a failure can be incorporated online into a cluster operation. A node can be incorporated into a cluster which has become unstable due to a failure using the gs_joincluster command. As a result of the node incorporation, the partitions will be rearranged autonomously and the node data and load balance will be adjusted.

In this way, although advance recovery preparations are not necessary in a single failure, recovery operations are necessary when operating in a single configuration or when there are multiple overlapping failures in the cluster configuration.

When operating in a cloud environment, even when physical disk failure or processor failure is not intended, there may be multiple failures such as a failure in multiple nodes constituting a cluster, or a database failure in multiple nodes.

### Type and treatment of failures

An overview of the failures which occur and the treatment method is shown in the table below.

A node failure refers to a situation in which a node has stopped due to a processor failure or an error in a GridDB server process, while a database failure refers to a situation in which an error has occurred in accessing a database placed in a disk.

| Configuration of GridDB | Type of failure           | Action and treatment                                                                                                                                                                                                                                                                                                                                                                               |
| ----------------------- | ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Single configuration    | Node failure              | Although access from the application is no longer possible, data in a transaction which has completed processing can be recovered simply by restarting the transaction, except when caused by a node failure. Recovery by another node is considered when the node failure is prolonged.                                                                                                           |
| Single configuration    | Database failure          | The database file is recovered from the backup data in order to detect an error in the application. Data at the backup point is recovered.                                                                                                                                                                                                                                                         |
| Cluster configuration   | Single node failure       | The error is covered up in the application, and the process can continue in nodes with replicas. Recovery operation is not necessary in a node where a failure has occurred.                                                                                                                                                                                                                       |
| Cluster configuration   | Multiple node failure     | If both owner/backup partitions of a replica exist in a failure target node, the cluster will operate normally even though the subject partitions cannot be accessed. Except when caused by a node failure, data in a transaction which has completed processing can be recovered simply by restarting the transaction. Recovery by another node is considered when the node failure is prolonged. |
| Cluster configuration   | Single database failure   | Since data access will continue through another node constituting the cluster when there is a database failure in a single node, the data can be recovered simply by changing the database deployment location to a different disk, and then starting the node again.                                                                                                                              |
| Cluster configuration   | Multiple database failure | A partition that cannot be recovered in a replica needs to be recovered at the point backup data is sampled from the latest backup data.                                                                                                                                                                                                                                                           |


### Client failover

If a node failure occurs when operating in a cluster configuration, the partitions (containers) placed in the failure node cannot be accessed. At this point, a client failover function to automatically connect to the backup node again and continue the process is activated in the client API. To automatically perform a failover countermeasure in the client API, the application developer does not need to be aware of the error process in the node.

However, due to a network failure or simultaneous failure of multiple nodes, an error may also occur and access to the target application operations may not be possible.

Depending on the data to be accessed, the following points need to be considered in the recovery process after an error occurs.

  - For a collection in which the timeseries container or row key is defined, the data can be recovered by executing the failed operation or transaction again.

  - For a collection in which the row key is not defined, the failed operation or transaction needs to be executed again after checking the contents of the DB.

[Note]
  - In order to simplify the error process in an application, it is recommended that the row key be defined when using a collection. If the data cannot be uniquely identified by a single column value but can be uniquely identified by multiple column values, a column having a value that links the values of the multiple columns is recommended to be set as the row key so that the data can be uniquely identified.

<a id="label_event_log"></a>
## Event log function

An event log is a log to record system operating information and messages related to event information e.g. exceptions which occurred internally in a GridDB node etc.

An event log is created with the file name gridstore-%Y%m%d-n.log in the directory shown in the environmental variable GS_LOG (Example: gridstore-20150328-5.log). 22/5000 The file switches at the following timing:

  - When the log is written first after the date changes
  - When the node is restarted
  - When the size of one file exceeds 1MB

The default value of the maximum number of event log files is 30. If it exceeds 30 files, it will be deleted from the old file. The maximum number can be changed with the node definition file.

Output format of event log is as follows.

  - (Date and time) (host name) (thread no.) (log level) (category) [(error trace no.): (error trace no. and name)] (message) < (base64 detailed information: Detailed information for problem  analysis in the support service)>
    
    An overview of the event can be found using the error trace number.

``` 

2014-11-12T10:35:29.746+0900 TSOL1234 8456 ERROR TRANSACTION_SERVICE [10008:TXN_CLUSTER_NOT_SERVICING] (nd={clientId=2, address=127.0.0.1:52719}, pId=0, eventType=CONNECT, stmtId=1) <Z3JpZF9zdG9yZS9zZXJ2ZXIvdHJhbnNhY3Rpb25fc2VydmljZS5jcHAgQ29ubmVjdEhhbmRsZXI6OmhhbmRsZUVycm9yIGxpbmU9MTg2MSA6IGJ5IERlbnlFeGNlcHRpb24gZ3JpZF9zdG9yZS9zZXJ2ZXIvdHJhbnNhY3Rpb25fc2VydmljZS5jcHAgU3RhdGVtZW50SGFuZGxlcjo6Y2hlY2tFeGVjdXRhYmxlIGxpbmU9NjExIGNvZGU9MTAwMDg=>

```

## Checking operation state

### Performance and statistical information

GridDB performance and statistical information can be checked in GridDB using the operating command gs_stat. gs_stat represents information common in the cluster and performance and statistical information unique to the nodes.

Among the outputs of the gs_stat command, the performance structure is an output that is related to the performance and statistical information.

An example of output is shown below. The output contents vary depending on the version.

```
-bash-4.1$ gs_stat -u admin/admin -s 192.168.0.1:10040
{
    ：
    "performance": {
        "batchFree": 0,
        "checkpointFileSize": 65536,
        "checkpointFileUsageRate": 0,
        "checkpointMemory": 2031616,
        "checkpointMemoryLimit": 1073741824,
        "checkpointWriteSize": 0,
        "checkpointWriteTime": 0,
        "currentTime": 1428024628904,
        "numConnection": 0,
        "numTxn": 0,
        "peakProcessMemory": 42270720,
        "processMemory": 42270720,
        "recoveryReadSize": 65536,
        "recoveryReadTime": 0,
        "sqlStoreSwapRead": 0,
        "sqlStoreSwapReadSize": 0,
        "sqlStoreSwapReadTime": 0,
        "sqlStoreSwapWrite": 0,
        "sqlStoreSwapWriteSize": 0,
        "sqlStoreSwapWriteTime": 0,
        "storeDetail": {
            "batchFreeMapData": {
                "storeMemory": 0,
                "storeUse": 0,
                "swapRead": 0,
                "swapWrite": 0
            },
            "batchFreeRowData": {
                "storeMemory": 0,
                "storeUse": 0,
                "swapRead": 0,
                "swapWrite": 0
            },
            "mapData": {
                "storeMemory": 0,
                "storeUse": 0,
                "swapRead": 0,
                "swapWrite": 0
            },
            "metaData": {
                "storeMemory": 0,
                "storeUse": 0,
                "swapRead": 0,
                "swapWrite": 0
            },
            "rowData": {
                "storeMemory": 0,
                "storeUse": 0,
                "swapRead": 0,
                "swapWrite": 0
            }
        },
        "storeMemory": 0,
        "storeMemoryLimit": 1073741824,
        "storeTotalUse": 0,
        "swapRead": 0,
        "swapReadSize": 0,
        "swapReadTime": 0,
        "swapWrite": 0,
        "swapWriteSize": 0,
        "swapWriteTime": 0,
        "syncReadSize": 0,
        "syncReadTime": 0,
        "totalLockConflictCount": 0,
        "totalReadOperation": 0,
        "totalRowRead": 0,
        "totalRowWrite": 0,
        "totalWriteOperation": 0
    },
    ：
}
```

Information related to performance and statistical information is explained below. The description of the storeDetail structure is omitted as this is internal debugging information.
  - The type is shown below.
      - CC: Current value of all cluster
      - c: Current value of specified node
      - CS: Cumulative value after service starts for all clusters
      - s: Cumulative value after service starts for all nodes
      - CP: Peak value after service starts for all clusters
      - p: Peak value after service starts for all nodes
  - Check the event figure to be monitored, and show the items that ought to be reviewed in continuing with operations.

| Output parameters          | Type | Description                                                                                                                                                                                                                                   | Event to be monitored                                                                                                                                                                                               |
| -------------------------- | ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| checkpointFileSize         | c    | Checkpoint file size (byte)                                                                                                                                                                                                                   |                                                                                                                                                                                                                     |
| checkpointFileUsageRate    | c    | Checkpoint file usage rate                                                                                                                                                                                                                    |                                                                                                                                                                                                                     |
| checkpointMemory           | c    | Checkpoint memory size for checkpoint use (byte)                                                                                                                                                                                              |                                                                                                                                                                                                                     |
| checkpointMemoryLimit      | c    | CheckpointMemoryLimit setting for checkpoint use (byte)                                                                                                                                                                                       |                                                                                                                                                                                                                     |
| checkpointWriteSize        | s    | CP file write size for checkpoint processing (byte)                                                                                                                                                                                           |                                                                                                                                                                                                                     |
| checkpointWriteTime        | s    | CP file write time for checkpoint processing (ms)                                                                                                                                                                                             |                                                                                                                                                                                                                     |
| checkpointFileAllocateSize | c    | The total size of allocated blocks in the checkpoint files (bytes)                                                                                                                                                                            |                                                                                                                                                                                                                     |
| currentTime                | c    | Current time                                                                                                                                                                                                                                  |                                                                                                                                                                                                                     |
| numConnection              | c    | Current no. of connections. Number of connections used in the transaction process, not including the number of connections used in the cluster process. Value is equal to the no. of clients + no. of replicas * no. of partitions retained. | If the no. of connections is insufficient in monitoring the log, review the connectionLimit value of the node configuration.                                                                                        |
| numSession                 | c    | Current no. of sessions                                                                                                                                                                                                                       |                                                                                                                                                                                                                     |
| numTxn                     | c    | Current no. of transactions                                                                                                                                                                                                                   |                                                                                                                                                                                                                     |
| peakProcessMemory          | p    | Peak value of the memory used in the GridDB server, including the storememory value which is the maximum memory size (byte) used in the process                                                                                               | If the peakProcessMemory or processMemory is larger than the installed memory of the node and an OS Swap occurs, additional memory or a temporary drop in the value of the storeMemoryLimit needs to be considered. |
| processMemory              | c    | Memory space used by a process (byte)                                                                                                                                                                                                         |                                                                                                                                                                                                                     |
| recoveryReadSize           | s    | Checkpoint file size read by the recovery process (byte)                                                                                                                                                                                      |                                                                                                                                                                                                                     |
| recoveryReadTime           | s    | Checkpoint file read time by the recovery processing (ms)                                                                                                                                                                                     |                                                                                                                                                                                                                     |
| sqlStoreSwapRead           | s    | Read count from the file by SQL store swap processing                                                                                                                                                                                         |                                                                                                                                                                                                                     |
| sqlStoreSwapReadSize       | s    | Read size from the file by SQL store swap processing (byte)                                                                                                                                                                                   |                                                                                                                                                                                                                     |
| sqlStoreSwapReadTime       | s    | Read time from the file by SQL store swap processing (ms)                                                                                                                                                                                     |                                                                                                                                                                                                                     |
| sqlStoreSwapWrite          | s    | Write count to the file by SQL store swap processing                                                                                                                                                                                          |                                                                                                                                                                                                                     |
| sqlStoreSwapWriteSize      | s    | Write size to the file by SQL store swap processing (byte)                                                                                                                                                                                    |                                                                                                                                                                                                                     |
| sqlStoreSwapWriteTime      | s    | Write time to the file by SQL store swap processing (ms)                                                                                                                                                                                      |                                                                                                                                                                                                                     |
| storeMemory                | c    | Memory space used in an in-memory database (byte)                                                                                                                                                                                             |                                                                                                                                                                                                                     |
| storeMemoryLimit           | c    | Memory space limit used in an in-memory database (byte)                                                                                                                                                                                       |                                                                                                                                                                                                                     |
| storeTotalUse              | c    | Full data capacity (byte) retained by the nodes, including the data capacity in the database file                                                                                                                                             |                                                                                                                                                                                                                     |
| swapRead                   | s    | Read count from the file by swap processing                                                                                                                                                                                                   |                                                                                                                                                                                                                     |
| swapReadSize               | s    | Read size from the file by swap processing (byte)                                                                                                                                                                                             |                                                                                                                                                                                                                     |
| swapReadTime               | s    | Read time from the file by swap processing (ms)                                                                                                                                                                                               |                                                                                                                                                                                                                     |
| swapWrite                  | s    | Write count to the file by swap processing                                                                                                                                                                                                    |                                                                                                                                                                                                                     |
| swapWriteSize              | s    | Write size to the file by swap processing (byte)                                                                                                                                                                                              |                                                                                                                                                                                                                     |
| swapWriteTime              | s    | Write time to the file by swap processing (ms)                                                                                                                                                                                                |                                                                                                                                                                                                                     |
| syncReadSize               | s    | Read size from the CP file by synchronous processing (byte)                                                                                                                                                                                   |                                                                                                                                                                                                                     |
| syncReadTime               | s    | Read time from the CP file by synchronous processing (ms)                                                                                                                                                                                     |                                                                                                                                                                                                                     |
| totalLockConflictCount     | s    | Row lock competing count                                                                                                                                                                                                                      |                                                                                                                                                                                                                     |
| totalReadOperation         | s    | Search process count                                                                                                                                                                                                                          |                                                                                                                                                                                                                     |
| totalRowRead               | s    | Row reading count                                                                                                                                                                                                                             |                                                                                                                                                                                                                     |
| totalRowWrite              | s    | Row writing count                                                                                                                                                                                                                             |                                                                                                                                                                                                                     |
| totalWriteOperation        | s    | Insert and update process count                                                                                                                                                                                                               |                                                                                                                                                                                                                     |

<a id="operating_commands"></a>
## Operating commands

The following commands are available in GridDB. All the operating command names of GridDB start with gs_.

| Type               | Command             | Functions                                                                                                                                                                                                            |
| ------------------ | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Node operations    | gs_startnode       | start node                                                                                                                                                                                                           |
|                    | gs_stopnode        | stop node                                                                                                                                                                                                            |
| Cluster operations | gs_joincluster     | Join a node to a cluster. Join to cluster configuration                                                                                                                                                              |
|                    | gs_leavecluster    | Cause a particular node to leave a cluster. Used, when causing a particular node to leave from a cluster for maintenance. The partition distributed to the node to leave the cluster will be rearranged (rebalance). |
|                    | gs_stopcluster     | Cause all the nodes, which constite a cluster, to leave the cluster. Used for stopping all the nodes. The partitions are not rebalanced when the nodes leave the cluster.                                            |                                                                                                                                                                                      |
|                    | gs_stat            | Get cluster data                                                                                                                                                                                                     |                                                                                                                                                                                  |
| User management    | gs_adduser         | Registration of administrator user                                                                                                                                                                                   |
|                    | gs_deluser         | Deletion of administrator user                                                                                                                                                                                       |
|                    | gs_passwd          | Change a password of an administrator user                                                                                                                                                                           |

---
[//]: # (<a id="label_parameters"></a>)
# Parameter

Describes the parameters to control the operations in GridDB. In the GridDB parameters, there is a node definition file to configure settings such as the setting information and usable resources etc., and a cluster definition file to configure operational settings of a cluster. Explains the meanings of the item names in the definition file and the settings and parameters in the initial state.

The unit of the setting is set as shown below.

  - The byte size can be specified in the following units: TB, GB, MB, KB, B, T, G, M, K, or lowercase notations of these units. Unit cannot be omitted unless otherwise stated.

  - Time can be specified in the following units: h, min, s, ms. Unit cannot be omitted unless otherwise stated.

　

## Cluster definition file (gs_cluster.json)

The same setting in the cluster definition file needs to be made in all the nodes constituting the cluster. As the partitionNum and storeBlockSize parameters are important parameters to determine the database structure, they cannot be changed after GridDB is started for the first time.

The meanings of the various settings in the cluster definition file are explained below.

By adding an item name, items that are not included in the initial state can be recognized by the system. Indicate whether the parameter can be changed and the change timing in the change field.

  - Disallowed: Node cannot be changed once it has been started. The database needs to be initialized if you want to change the setting.
  - Restart: Parameter can be changed by restarting all the nodes constituting the cluster.
  - Online: Parameters that are currently in operation online can be changed. However, the contents in the definition file need to be manual amended as the change details will not be perpetuated.

　

| Configuration of GridDB                      | Default   | Meaning of parameters and limitation values                                                                                                                                                                                                                                                               | Change     |
| -------------------------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| /notificationAddress                         | 239.0.0.1 | Standard setting of a multi-cast address. This setting will become valid if a parameter with the same cluster, transaction name is omitted. If a different value is set, the address of the individual setting is valid.                                                                                  | Restart    |
| /dataStore/partitionNum                      | 128       | Specify a common multiple that will allow the number of partitions to be divided and placed by the number of constituting clusters. Integer: Specify an integer that is 1 or higher and 10000 or lower.                                                                                                   | Disallowed |
| /dataStore/storeBlockSize                    | 64KB      | Specify the disk I/O size from 64KB,1MB,4MB,8MB,16MB,32MB. Larger block size enables more records to be stored in one block, suitable for full scans of large tables, but also increases the possibility of conflict. Select the size suitable for the system. Cannot be changed after server is started. | Disallowed |
| /cluster/clusterName                         | -        | Specify the name for identifying a cluster. Mandatory input parameter.                                                                                                                                                                                                                                    | Restart    |
| /cluster/replicationNum                      | 2         | Specify the number of replicas. Partition is doubled if the number of replicas is 2.                                                                                                                                                                                                                      | Restart    |
| /cluster/notificationAddress                 | 239.0.0.1 | Specify the multicast address for cluster configuration                                                                                                                                                                                                                                                   | Restart    |
| /cluster/notificationPort                    | 20000     | Specify the multicast port for cluster configuration. Specify a value within a specifiable range as a multi-cast port no.                                                                                                                                                                                 | Restart    |
| /cluster/notificationInterval                | 5s        | Multicast period for cluster configuration. Specify the value more than 1 second and less than 2<sup>31</sup> seconds.                                                                                                                                                                                    | Restart    |
| /cluster/heartbeatInterval                   | 5s        | Specify a check period (heart beat period) to check the node survival among clusters. Specify the value more than 1 second and less than 2<sup>31</sup> seconds.                                                                                                                                          | Restart    |
| /cluster/loadbalanceCheckInterval            | 180s      | To adjust the load balance among the nodes constituting the cluster, specify a data sampling period, as a criteria whether to implement the balancing process or not. Specify the value more than 1 second and less than 2<sup>31</sup> seconds.                                                          | Restart    |
| /cluster/notificationMember                  | -        | Specify the address list when using the fixed list method as the cluster configuration method.                                                                                                                                                                                                            | Restart    |
| /cluster/notificationProvider/url            | -        | Specify the URL of the address provider when using the provider method as the cluster configuration method.                                                                                                                                                                                               | Restart    |
| /cluster/notificationProvider/updateInterval | 5s        | Specify the interval to get the list from the address provider. Specify the value more than 1 second and less than 2<sup>31</sup> seconds.                                                                                                                                                                | Restart    |
| /sync/timeoutInterval                        | 30s       | Specify the timeout time during data synchronization among clusters. 　If a timeout occurs, the system load may be high, or a failure may have occurred. Specify the value more than 1 second and less than 2<sup>31</sup> seconds.                                                                        | Restart    |
| /transaction/notificationAddress             | 239.0.0.1 | Multi-cast address that a client connects to initially. Master node is notified in the client.                                                                                                                                                                                                            | Restart    |
| /transaction/notificationPort                | 31999     | Multi-cast port that a client connects to initially. Specify a value within a specifiable range as a multi-cast port no.                                                                                                                                                                                  | Restart    |
| /transaction/notificationInterval            | 5s        | Multi-cast period for a master to notify its clients. Specify the value more than 1 second and less than 2<sup>31</sup> seconds.                                                                                                                                                                          | Restart    |
| /transaction/replicationMode                 | 0         | Specify the data synchronization (replication) method when updating the data in a transaction. Specify a string or integer, "ASYNC"or 0 (non-synchronous), "SEMISYNC"or 1 (quasi-synchronous).                                                                                                            | Restart    |
| /transaction/replicationTimeoutInterval      | 10s       | Specify the timeout time for communications among nodes when synchronizing data in a quasi-synchronous replication transaction. Specify the value more than 1 second and less than 2<sup>31</sup> seconds.                                                                                                | Restart    |
| /transaction/authenticationTimeoutInterval   | 5s        | Specify the authentication timeout time.                                                                                                                                                                                                                                                                  | Restart    |
| /sql/notificationAddress                     | 239.0.0.1 | Multi-cast address when the JDBC client is connected initially. Master node is notified in the client.                                                                                                                                                                                               | Restart    |
| /sql/notificationPort                        | 41999     | Multi-cast port when the JDBC client is connected initially. Specify a value within a specifiable range as a multi-cast port no.                                                                                                                                                                     | Restart    |
| /sql/notificationInterval                    | 5s        | Multi-cast period for a master to notify its JDBC clients. Specify the value more than 1 second and less than 2<sup>31</sup> seconds.                                                                                                                                                                | Restart    |

　

## Node definition file (gs_node.json)

A node definition file defines the default settings of the resources in nodes constituting a cluster. In an online operation, there are also parameters whose values can be changed online from the resource, access frequency, etc., that have been laid out. Conversely, note that there are also values (concurrency) that cannot be changed once set.

The meanings of the various settings in the node definition file are explained below.

By adding an item name, items that are not included in the initial state can be recognized by the system. Indicate whether the parameter can be changed and the change timing in the change field.
  - Disallowed: Node cannot be changed once it has been started. The database needs to be initialized if you want to change the setting.
  - Restart: Parameter can be changed by restarting all the nodes  constituting the cluster.
  - Online: Parameters that are currently in operation online can be changed. However, the contents in the definition file need to be manual amended as the change details will not be perpetuated.

Specify the directory by specifying the full path or a relative path from the GS_HOME environmental variable. For relative path, the initial directory of GS_HOME serves as a reference point. Initial configuration directory of GS_HOME is /var/lib/gridstore.

| Configuration of GridDB                         | Default                     | Meaning of parameters and limitation values     | Change       |
|--------------------------------------|----------------------------|---------------------------------|-----|
| /serviceAddress                      | -                       | Set the initial value of each cluster, transaction, sync service address. The initial value of each service address can be set by setting this address only without having to set the addresses of the 3 items.  | Restart       |
| /dataStore/dbPath                    | data                       | The deployment directory of the database file is specified by the full path or a relative path   | Restart       |
| /dataStore/dbFileSplitCount          | 0 (no splitting)              | Number of checkpoint file splitting   | Disallowed      |
| /dataStore/dbFilePathList            | Empty list                  | The list of directories where the split checkpoint files are placed when the checkpoint file is to be split.<br /> Required if 1 or more is specified as dbFileSplitCount. More than one can be specified (example: ["/stg01", "/stg02"]). Except that, the number of directories greater than dbFileSplitCount cannot be specified.   | Restart   |
| /dataStore/backupPath                | backup                     | Specify the backup file deployment directory path.                                  | Restart       |
| /dataStore/syncTempPath              | sync                       | Specify the path of the Data sync temporary file directory.                                  | Restart       |
| /dataStore/storeMemoryLimit          | 1024MB                     | Upper memory limit for data management                                                       | Online |
| /dataStore/concurrency               | 4                          | Specify the concurrency of processing.                                                                        | Disallowed       |
| /dataStore/logWriteMode              | 1                          | Specify the log writing mode and cycle. If the log writing mode period is -1 or 0, log writing is performed at the end of the transaction. If it is 1 or more and less than 2<sup>31</sup>, log writing is performed at a period specified in seconds  | Restart       |
| /dataStore/persistencyMode           | 1(NORMAL)                  | In the perpetuation mode, the period that the update log file is maintained during a data update is specified. Specify either 1 (NORMAL) or 2 (RETAINING_ALL_LOGS). For "NORMAL", a transaction log file which is no longer required will be deleted by the checkpoint. For "RETAINING_ALL_LOGS", all transaction log files are retained. | Restart       |
| /dataStore/storeWarmStart            | false(invalid)                | Specify whether to save in-memory up to the upper limit of the chunk memory during a restart.                        | Restart       |
| /dataStore/affinityGroupSize         | 4                          | Number of affinity groups                                                        | Restart       |
| /dataStore/storeCompressionMode      | NO_COMPRESSION   | Data block compression mode                                                                | Restart       |
| /dataStore/autoExpire                | false                      | Specify whether to delete the rows of a container in which an expiry release is set automatically after the rows become cold data. false: Not delete automatically (Needs to be deleted by executing the long term archive) true: Delete automatically                                                                                                            | Online|
| /checkpoint/checkpointInterval       | 60s                       | Checkpoint process execution period to perpetuate a data update block in the memory        | Restart       |
| /checkpoint/checkpointMemoryLimit    | 1024MB                     | Upper limit of special checkpoint write memory* Pool the required memory space up to the upper limit when there is a update transaction in the checkpoint.                                                                                                                                    | Online |
| /checkpoint/useParallelMode          | false(invalid)                | Specify whether to execute the checkpoint concurrently. *The no. of concurrent threads is the same as the concurrency.| Restart       |
| /checkpoint/checkpointCopyInterval   | 100ms                      | Output process interval when outputting a block with added or updated data to a disk in a checkpoint process.        | Restart       |
| /cluster/serviceAddress              | Comforms to the upper serviceAddress | Standby address for cluster configuration                                            | Restart       |
| /cluster/servicePort                 | 10010                      | Standby port for cluster configuration                                             | Restart       |
| /cluster/notificationInterfaceAddress  | ""                       | Specify the address of the interface which sends multicasting packets. | Restart       |
| /sync/serviceAddress                 | Comforms to the upper serviceAddress | Reception address for data synchronization among the clusters                                 | Restart       |
| /sync/servicePort                    | 10020                      | Standby port for data synchronization                                              | Restart       |
| /system/serviceAddress               | Comforms to the upper serviceAddress | Standby address for operation commands                                           | Restart       |
| /system/servicePort                  | 10040                      | Standby port for operation commands                                            | Restart       |
| /system/eventLogPath                 | log                        | Event log file deployment directory path                            | Restart       |
| /transaction/serviceAddress          | Comforms to the upper serviceAddress | Standby address for transaction processing for client communication, used also for cluster internal communication when /transaction/localserviceAddress is not specified.  | Restart       |
| /transaction/localServiceAddress     | Comforms to the upper serviceAddress | Standby address for transaction processing for cluster internal communication  | Restart       |
| /transaction/servicePort             | 10001                      | Standby port for transaction process                                    | Restart       |
| /transaction/connectionLimit         | 5000                       | Upper limit of the no. of transaction process connections                                       | Restart       |
| /transaction/transactionTimeoutLimit | 300s                      | Transaction timeout upper limit                             | Restart       |
| /transaction/reauthenticationInterval  | 0s(disabled)                 | Re-authentication interval. (After the specified time has passed, authentication process runs again and updates permissions of the general users who have already been connected.) The default value, 0 sec, indicates that re-authentication is disabled.| Online       |
| /transaction/workMemoryLimit         | 128MB                      | Maximum memory size for data reference (get, TQL) in transaction processing (for each concurrent processing)       | Online       |
| /transaction/notificationInterfaceAddress | ""                    | Specify the address of the interface which sends multicasting packets.                | Restart       |
| /sql/serviceAddress                  | Comforms to the upper serviceAddress | Standby address for NewSQL I/F access processing for client communication, used also for cluster internal communication when / /sql/localServiceAddress is not specified.   | Restart       |
| /sql/localServiceAddress             | Comforms to the upper serviceAddress | Standby address for NewSQL I/F access processing for cluster internal communication   | Restart       |
| /sql/servicePort                     | 20001                      | Standby port for New SQL access process                             | Restart       |
| /sql/storeSwapFilePath               | swap                       | SQL intermediate store swap file directory                 | Restart       |
| /sql/storeSwapSyncSize               | 1024MB                     | SQL intermediate store swap file and cache size  | Restart       |
| /sql/storeMemoryLimit                | 1024MB                     | Upper memory limit for intermediate data held in memory by SQL processing.                                 | Restart       |
| /sql/workMemoryLimit                 | 32MB                       | Upper memory limit for operators in SQL processing                                   | Restart       |
| /sql/workCacheMemory                 | 128MB                      | Upper size limit for cache without being released after use of work memory.  | Restart       |
| /sql/connectionLimit                 | 5000                       | Upper limit of the no. of connections processed for New SQL access                                      | Restart       |
| /sql/concurrency                     | 4                          | No. of simultaneous execution threads                                                         | Restart       |
| /sql/traceLimitExecutionTime         | 300s                      | The lower limit of execution time of a query to write in an event log                           | Online |
| /sql/traceLimitQuerySize             | 1000                       | The upper size limit of character strings in a slow query (byte)                         | Online |
| /sql/notificationInterfaceAddress    | ""                         | Specify the address of the interface which sends multicasting packets.                       | Restart       |
| /trace/fileCount                     | 30                         | Upper file count limit for event log files.                                                 | Restart       |




# System limiting values


## Limitations on numerical value


| Block size                                                              | 64KB        | 1MB - 32MB               |
| ----------------------------------------------------------------------- | ----------- | ------------------------ |
| STRING/GEOMETRY data size                                               | 31KB        | 128KB                    |
| BLOB data size                                                          | 1GB - 1Byte | 1GB - 1Byte              |
| Array length                                                            | 4000        | 65000                    |
| No. of columns                                                          | 1024        | Approx. 7K - 32000 (*1) |
| No. of indexes (Per container)                                          | 1024        | 16000                    |
| No. of columns subject to linear complementary compression              | 100         | 100                      |
| No. of users                                                            | 128         | 128                      |
| No. of databases                                                        | 128         | 128                      |
| URL of trigger                                                          | 4KB         | 4KB                      |
| Number of affinity groups                                               | 10000       | 10000                    |
| No. of divisions in a timeseries container with a cancellation deadline | 160         | 160                      |
| Size of communication buffer managed by a GridDB node                   | Approx. 2GB | Approx. 2GB              |

| Block size     | 64KB        | 1MB          | 4MB           | 8MB           | 16MB        | 32MB        |
| -------------- | ----------- | ------------ | ------------- | ------------- | ----------- | ----------- |
| Partition size | Approx. 4TB | Approx. 64TB | Approx. 256TB | Approx. 512TB | Approx. 1PB | Approx. 2PB |

  - STRING, URL of trigger
      - Limiting value is equivalent to UTF-8 encode
  - Spatial-type
      - Limiting value is equivalent to the internal storage format
  - (*1) The number of columns
      - There is a restriction on the upper limit of the number of columns. The total size of a fixed length column (BOOL, INTEGER, FLOAT, DOUBLE, TIMESTAMP type) must be less than or equal to 59 KB. The upper limit of the number of columns is 32000 if the type is not a fixed length column.
          - Example) If a container consists of LONG type columns: the upper limit of the number of columns is 7552 ( The total size of a fixed length column 8B * 7552 = 59KB )
          - Example) If a container consists of BYTE type columns: the upper limit of the number of columns is 32000 ( The total size of a fixed length column 1B * 32000 = Approx. 30KB -&gt; Up to 32000 columns can be created because the size restriction on a fixed length column does not apply to it )
          - Example) If a container consists of STRING type columns: the upper limit of the number of columns is 32000 ( Up to 32000 columns can be created because the size restriction on a fixed length column does not apply to it )

## Limitations on naming


| Field                   | Allowed characters                                     | Maximum length                    |
|------------------------|----------------------------------------------------|-------------------------------|
| Administrator user             | The head of name is "gs#" and the following characters are either alphanumeric or '_' | 64characters                        |
| General user             | Alphanumeric, '_', '-', '.', '/', and '='                   | 64characters                        |
| &lt;Password&gt;             | Composed of an arbitrary number of characters<br /> using the unicode code point | 64 bytes (by UTF-8 encoding) |
| cluster name             | Alphanumeric, '_', '-', '.', '/', and '='                   | 64 characters                        |
| Database name         | Alphanumeric, '_', '-', '.', '/', and '='                   | 64 characters                        |
| Container name<br /> Table name<br /> View name | Alphanumeric, '_', '-', '.', '/', and '='<br /> (and '@' only for specifying a node affinity) | 16384 characters (for 64KB block)<br /> 131072 characters (for 1MB - 32MB block) |
| Column name              | Alphanumeric, '_', '-', '.', '/', and '='                   | 256 characters                       |
| Index name                 | Alphanumeric, '_', '-', '.', '/', and '='                   | 16384 characters (for 64KB block)<br /> 131072 characters (for 1MB - 32MB block) |
| Trigger name               | Alphanumeric, '_', '-', '.', '/', and '='                   | 256 characters                       |
| Backup name         | Alphanumeric and '_'                                       | 12 characters                        |
| Data Affinity     | Alphanumeric, '_', '-', '.', '/', and '='                   | 8 characters     

  - Case sensitivity 
      - Cluster names, trigger names, backup names and passwords are case-sensitive. So the names of the following example are handled as different names.
        
      ``` 
      Example) trigger, TRIGGER
      ```

  - Other names are not case-sensitive. Uppercase and lowercase characters are identified as the same.
  - Uppercase and lowercase characters in names at the creation are hold as data.
  - The names enclosed with '"' in TQL or SQL are case-sensitive. In that case, uppercase and lowercase characters are not identified as the same.
    
  ```   
  Example) Search on the container "SensorData" and the column "Column1" 
  select "Column1" from "SensorData"   Success 
  select "COLUMN1" from "SENSORDATA"   Fail (Because "SENSORDATA" container does not exist)
  ```

  - Specifying names by TQL and SQL
      - In the case that the name is not enclosed with '"', it can contain only alphanumeric and '_'. To use other characters, the name is required to be enclosed with '"'.  
      ``` 
      Example) select "012column", data_15 from "container.2017-09"
      ```
