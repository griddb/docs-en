# GridDB Quickstart Guide

## Table of Contents
* [Introduction](#Introduction)
* [Using source code](#Using-source-code)
* [Using RPM](#Using-RPM)
---
# Introduction
This manual explains how to get started with GridDB. You will learn how to install GridDB, start and stop GridDB node, and data manipulation.

In a nutshell, you can install GridDB on one machine and start / stop GridDB node and execute a series of procedures to execute the sample program.

1. [Using source code](#using-source-code)
1. [Using RPM](#using-rpm)
---
# Using source code

We have confirmed the operation on CentOS 7.6 (gcc 4.8.5).

## Build a server and client(Java)

Download the GridDB source code package build to build the nodes and clusters. And set three environment variable.

```
    $ git clone https://github.com/griddb/griddb.git
    $ cd griddb
    $ ./bootstrap.sh
    $ ./configure
    $ make

    $ export GS_HOME=$PWD
    $ export GS_LOG=$PWD/log
    $ export PATH=${PATH}:$GS_HOME/bin
````
The following environment variables are defined.


| Environment variables | Value | Meaning |
|-----------------------|-------|---------|
| GS_HOME | Directory where source code file is decompressed | GridDB home directory |
| GS_LOG | $GS_HOME/log | Event log file output directory |

#### :warning: Note
- These environment variables are referenced by the operational commands shown in the following subsections.
- Default building environment repeals the trigger function. Add the following option in build to enable a trigger function. Add the following option in build to enable the trigger function.
```
$ ./configure --enable-activemq
```

## Environmental settings

### Network settings

GridDB uses multicast communication to constitute a cluster.
Set the network to enable multicast communication.

First, check the host name and an IP address.
Execute "hostname -i" command to check the settings of an IP address of the host.
If the IP address of the machine is the same as below, no need to perform additional network setting and you can jump to the next sectiion. 

[Example]
```
$ hostname -i
192.168.11.10
```

If the following message or the loopback addresses points to 127.0.0.1, additional network configuration is needed. .

[Example]
```
$ hostname -i 
hostname: no IP address was found for this hostname.
```
or
```
$ hostname -i
127.0.0.1
```

Follow the following procedures from 1 to 3.
　
#### Network settings

1. Check the host name and OS IP address.

   [Example]
   ```
   $ hostname
   MY_HOST
   ```

1. Check the IP address.

   [Example]
   ```
   $ ip -f inet -o addr show eth0 | cut -d' ' -f 7 | cut -d/ -f 1
   192.168.11.10
   ```

2. Add the host name and IP address,  
   which were checked in the procedure 1, to the /etc/hosts file as a root user.

   [Example]
   ```
   # cd /
   # pwd
   /root
   # echo "192.168.11.10 MY_HOST" >> /etc/hosts
   ```

3. Execute "hostname -i" command to check the settings of an IP address  of the host.  Check that the IP address specified in the procedure 2 is displayed.


   [Example]
   ```
   $ hostname -i
   192.168.11.10
   ```

### GridDB Admin user settingr

An admin user is used for authentication purposes in nodes and clusters. 
Administrator user information is stored in the *User definition file*. The default file is as shown below.

$GS_HOME/conf/password

The following default users exist just after installation.

| User | Password |
|------|----------|
| admin | No settings |

Administrator user information including the above-mentioned default users can be changed using the user administration command in the operating commands.

| Command | Function |
|---------|----------|
| gs_adduser | Add an administrator user |
| gs_deluser | Delete an administrator user |
| gs_passwd | Change the password of an administrator user |


Change the password as shown below when using a default user.
The password is encrypted during registration.

#### :warning: Note
- Default user password has not been set. Be sure to change the password as the server will not start if the administrator user password is not set.
   ```
   $ gs_passwd admin
   Password:（Input password）
   Retype password:（Input password again）
   ```
- When adding a new administrator user except a default user, the user name has to start with gs#.

- One or more ASCII alphanumeric characters and the underscore sign “_” can be used after gs#.

- An example on adding a new administrator user is shown below.

   ```
   $ gs_adduser gs#newuser
   Password:（Input password）
   Retype password:（Input password again）
   ```

### Setting parameters of a node

To operate GridDB after installation, initial setting of parameters, such as addresses and the cluster name, is required.
In this manual, only "Cluster name", an essential item, is set and for other items default values are used.

Describe "Cluster name" of a cluster in the cluster definition file.
The cluster definition file is "$ GS_HOME / conf / gs_cluster.json".

Enter the cluster name in the "" clusterName ":" "" part.
Use the name "myCluster" here.

[Example of description in the file]
```
$ vi $GS_HOME/conf/gs_cluster.json

{
        "dataStore":{
                "partitionNum":128,
                "storeBlockSize":"64KB"
        },
        "cluster":{
                "clusterName":"myCluster",
                "replicationNum":2,
                "notificationAddress":"239.0.0.1",
                "notificationPort":20000,
                "notificationInterval":"5s",
                "heartbeatInterval":"5s",
                "loadbalanceCheckInterval":"180s"
        },
        "sync":{
                "timeoutInterval":"30s"
        },
        "transaction":{
                "notificationAddress":"239.0.0.1",
                "notificationPort":31999,
                "notificationInterval":"5s",
                "replicationMode":0,
                "replicationTimeoutInterval":"10s"
        }
}
```

##### :memo: Note
- Using a cluster name, which is unique on the sub network is recommended.
- A cluster name must be composed of one or more ASCII alphanumeric characters or the underscore "\_". A number is not accepted as the first character of the name. The name is also not case-sensitive. The maximum length of the name must be 64 characters.

### Explanation of terms

Except for network settings and the cluster name, GridDB can operate with the default setting values. The main setting items are shown below. Use the values below when executing the tools in the following sections.

| Setting items | Value |
|---------------|-------|
| IP address | The IP address displayed when executing "hostname -i" command |
| Cluster name | myCluster |
| Multicast address | 239.0.0.1 (Default value) |
| Multicast port number | 31999 (Default value) |
| User name | admin (Default value) |
| User password | admin |
| SQL multicast address | 239.0.0.1 (Default value) |
| SQL multicast port number | 41999 (Default value) |

　
## Starting/stopping

Start/stop a GridDB node and a cluster. Starting/stopping operations using operating commands are introduced from among some operational methods of GridDB. The operation command is executed as a gsadm user.

A list of operating commands

| Command | Function |
|---------|----------|
| gs_startnode | Starting a node |
| gs_joincluster | Add a node to a cluster. |
| gs_leavecluster | Detach a node from a cluster |
| gs_stopcluster | Stopping a cluster |
| gs_stopnode | Stop a node (shutdown) |
| gs_stat | Get node internal data |

#### :warning: Note
- If a proxy variable (http_proxy) has been set up, set the node address in no_proxy and exclude it from the proxy target.
   Otherwise, since the operating command uses REST/http communications, it will connect to the proxy server and will fail to operate.
   Example of setting
   ```
   $ export no_proxy=localhost,127.0.0.1,192.168.11.10
   ```

### Starting operations

After installing and setting up the GridDB node, the flow for starting the GridDB cluster is as follows:
1. Starting a node
2. Starting a cluster

As explained in the "node and cluster" section, GridDB cluster is formed and  the cluster service is started when the number of nodes specified by the user joins the cluster.
Cluster service will not be started and access to the cluster from the application will not be possible until all the nodes constituting a cluster have joined the cluster.

The example in this manual is "single configuration" using one node. Start up a node, add the node to a cluster and then start the cluster.

#### Starting a node

Use gs_startnode command to start a node. 
Specify the user name and the password of admin, the management user, along with the user authentication option -u, and specify -w option to wait for the node to start.

[Example]
```
$ gs_startnode -u admin/admin -w
```

#### Starting a cluster

Execute the gs_joincluster command to join the cluster at each node and start the cluster.  Specify the user name and password of admin, the management user, along with the user authentication option -u, and specify -w option to wait until the starting up of the cluster. Specify a cluster name with -c option. .

[Example]
```
$ gs_joincluster -u admin/admin –w -c myCluster
```

After starting the cluster, check the cluster status using the gs_stat command.   
Specify the user name and the password of admin, the management user, with the user authentication option -u. To check the status of a cluster, extract the lines including "Status" using grep command.

[Example]
```
$ gs_stat -u admin/admin | grep Status
        "clusterStatus": "MASTER",
        "nodeStatus": "ACTIVE",
        "partitionStatus": "NORMAL"
```

If the three status displayed by "gs_stat" command are the same as above, the cluster has started normally. The cluster is accessible from an application.

### Stopping operations

#### Basic flow

The stopping procedures of GridDB is as follows.
1. Stopping a cluster
2. Stopping a node

Contrary to the startup procedures, stop the cluster first and then stop the each node. When the cluster is stopped, it becomes inaccessible from the applications.

#### Stopping a cluster

Execute the following cluster stop command. When the cluster stop command is executed, the cluster becomes inaccessible from the applications.

- gs_stopcluster -u admin/admin -w

   - Specify the user name and the password of admin, the management user, along with the user authentication option -u, and specify -w option to wait until the cluster has stopped.

[Example]
```
$ gs_stopcluster -u admin/admin -w
.
.
The GridDB cluster has been stopped.
```

#### Stopping a node

To stop the node, execute the gs_stopcluster command. When the cluster stop command is executed, the cluster becomes inaccessible from the applications.
Specify the user name and the password of admin, the management user, along with the user authentication option -u, and specify -w option to wait until the cluster has stopped.
- gs_stopnode -u admin/admin -w

   - Specify the user name and the password of admin, the management user, along with the user authentication option -u, and specify -w option to wait until the node is stopped.

[Example]
```
$ gs_stopnode -u admin/admin -w
The GridDB node is stopped.
.
The GridDB node has been stopped.
```

## Build/execution method

An example on how to build and execute a program is as shown.

[For Java]

1. Setting the environment variable CLASSPATH
2. Copy the sample program to the gsSample directory
3. Build
4. Run

```
$ export CLASSPATH=${CLASSPATH}:$GS_HOME/bin/gridstore.jar:.
$ mkdir gsSample
$ cp $GS_HOME/docs/sample/program/Sample1.java gsSample/.
$ javac gsSample/Sample1.java
$ java gsSample/Sample1 239.0.0.1 31999 setup_cluster_name admin your_password
```
---
# Using RPM

We have confirmed the operation on CentOS 7.6.

#### :warning: Note
- When you install this package, a gsadm OS user are created in the OS. Execute the operating command as the gsadm user.   
   Example
   ```
   # su - gsadm
   $ pwd
   /var/lib/gridstore
   ```
- You do not need to set environment variable GS_HOME and GS_LOG. And the place of operating command is set to environment variable PATH.
- There is Java client library (gridstore.jar) on /usr/share/java and a sample on /usr/gridb-XXX/docs/sample/programs.

## Install

Install the package of your target OS.
	
	(CentOS)
    $ sudo rpm -ivh griddb_nosql-X.X.X-linux.x86_64.rpm
    
	X.X.X means version
	
### User and directory structure after installation

When the GridDB package is installed, the following users and directory structure will be created.

#### GridDB users and group

The OS group gridstore and user gsadm are created. Use the user gsadm as the operator of GridDB.

| User | Group |  GridDB home directory path |
|------|-------|-----------------------------|
| gsadm | gridstore | /var/lib/gridstore |

The following environment variables are defined for user gsadm (in the .bash_profile file):

| Environment variables | Value | Meaning |
|---------|----|------|
| GS_HOME | /var/lib/gridstore | User gsadm/GridDB home directory |
| GS_LOG | /var/lib/gridstore/log | The output directory of the event log file of a node |



#### Directory hierarchy

The following two directories are created: GridDB home directory which contains files such as a node definition file and database files, the installation directory which contains the installed files.

###### GridDB home directory
```
/var/lib/gridstore/                      #GridDB home directory
                   conf/                 # Definition file directory
                        gs_cluster.json  #Cluster definition file
                        gs_node.json     #Node definition file
                        password         #User definition file
                   data/                 # Database file directory
                   log/                  # Log directory
```

###### Installation directory
```
Installation directory
            bin/                        # Operation command, module directory
            conf/                       #Definition file directory
                gs_cluster.json         # Custer definition file
                gs_node.json            #Node definition file
                password                #User definition file
            3rd_party/                  
            docs/
                manual/
                sample/
```

## Environmental settings

[When using source code](#using-source-code)the "[environmental setting ](#environmental-setting)" is the same。

## Starting/stopping

Operate as the gsadm user. Other than that, it is the same as "Start / Stop" in "[When using source code]".

## Build/execution method

An example on how to build and execute a program is as shown.

[For Java]

1. Setting environment variables
2. Copy the sample program to the gsSample directory
3. Build
4. Run

```
$ export CLASSPATH=${CLASSPATH}:/usr/share/java/gridstore.jar:.
$ mkdir gsSample
$ cp /usr/gridstore-X.X.X/docs/sample/program/Sample1.java gsSample/.
$ javac gsSample/Sample1.java
$ java gsSample/Sample1 239.0.0.1 31999 setup_cluster_name admin your_password
```


## GridDB uninstallation

If you no longer need GridDB, uninstall the package. Execute the following procedure with root authority.

[Example]

    (CentOS)
    $ sudo rpm -e griddb_nosql

#### :warning: Note
- Files under the GridDB home directory such as definition files and data files will not be uninstalled. If you do not need it, delete it manually.

---
