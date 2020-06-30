
# Installing and configuring Hadoop Cluster on Debian

## 1) Pre-Configuration:
VirtualBox programm with standard installed Debian Operation Sytem, standard login root and password.
After login into system we need to do next steps:
##### apt-get update && apt-get upgrade
##### apt-get install ssh
##### apt-get install rsync
##### apt-get install net-tools
##### apt-get install dnsutils

## 2) Then in Debian OS we need to install Java Virtual Environment for this we need just to type a command: 
##### apt-get install default-jre
after finishing installation we can see where was authomatically installed Java JVE :
##### update-alternatives --display java
result will be:

link current points to /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java




## 3) Adding new group and new user for Hadoop:

##### sudo addgroup hadoop
##### sudo adduser --ingroup hadoop hduser
##### sudo usermod -aG sudo hduser

## 4) Installing key for ssh
First of all we need to generate new ssh key:

##### ssh-keygen -t rsa -P ""


During geneation it will be asked to install a password. For test environment we can made it without password.

Next step it is to add genereted key in the list of authorizated keys, following command will do it:

##### cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys

after geneation we will check connection to localhost 
##### ssh localhost 


### 4.1) We are downloading HADOOP with a command
##### cd downloads/
##### wget http://apache-mirror.rbc.ru/pub/apache/hadoop/common/stable/hadoop-2.9.2.tar.gz   

Unpackiging it into: /usr/local/, rename folder and give to HDUSER the owner rights:

##### sudo mv /downloads/hadoop-2.9.2.tar.gz /usr/local/
##### cd /usr/local/
##### sudo tar xzf #### hadoop-2.9.2.tar.gz   
##### sudo mv hadoop-2.9.2 hadoop
##### chown -R hduser:hadoop hadoop



## 5) in Debian OS we need to add Hadoop variables в bashrc 

##### use a command nano /.bashrc:

#HADOOP VARIABLES START

export HADOOP_INSTALL=/usr/local/hadoop/

export PATH=$PATH:$HADOOP_INSTALL/bin

export PATH=$PATH:$HADOOP_INSTALL/sbin

export HADOOP_MAPRED_HOME=$HADOOP_INSTALL

export HADOOP_COMMON_HOME=$HADOOP_INSTALL

export HADOOP_HDFS_HOME=$HADOOP_INSTALL

export YARN_HOME=$HADOOP_INSTALL

#HADOOP VARIABLES END



## 6) In hadoop-env.sh we must add route to JAVA_HOME:

##### export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-am64/jre

Ignore Note: You may encounter with the error, “WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform… using builtin-java classes where applicable” if you don’t enter the second line as given above in hadoop-env.sh file.
## 7) We need to add in core-site.xml:

Enter the following content in between the <configuration></configuration> tag:

 <property>

  <name>hadoop.tmp.dir</name>

  <value>/app/hadoop/tmp</value>

  <description>A base for other temporary directories.</description>

 </property>

 <property>

  <name>fs.default.name</name>

  <value>hdfs://master:9000</value>

  <description>The name of the default file system.  A URI whose

  scheme and authority determine the FileSystem implementation.  The

  uri’s scheme determines the config property (fs.SCHEME.impl) naming

  the FileSystem implementation class.  The uri’s authority is used to

  determine the host, port, etc. for a filesystem.</description>

 </property>
 
## 8) Configuration /usr/local/hadoop/etc/hadoop/mapred-site.xml
This file is used to specify which framework is being used for MapReduce. By default, the folder /usr/local/hadoop/etc/hadoop/ contains the file named mapred-site.xml.template which has to be duplicated and named as mapred-site.xml

To duplicate the file,

##### sudo cp /usr/local/hadoop/etc/hadoop/mapred-site.xml.template /usr/local/hadoop/etc/hadoop/mapred-site.xml


## 9)

<configuration>
            <property>
                       <name>mapreduce.framework.name</name>
                       <value>yarn</value>
            </property>
</configuration>

## 10) Setting for HDFS lokated in etc/hadoop/hdfs-site.xml:
We need to create special folder with "tmp" name.
Here parameter dfs.replication indicates replication parameter, which will be stored on file system. For 1 master and two slaves we indicate the number 3.
This number will be no more then components in cluster.

Parameters dfs.namenode.name.dir and dfs.datanode.data.dir gives the roots, where can physicaly placed data and information in HDFS. 



<configuration>
            <property>
                       <name>dfs.replication</name>
                       <value>1</value>
            </property>
            <property>
                       <name>dfs.namenode.name.dir</name>
                       <value>file:/usr/local/hadoop/tmp/hdfs/namenode</value>
            </property>
            <property>
                       <name>dfs.datanode.data.dir</name>
                       <value>file:/usr/local/hadoop/tmp/hdfs/datanode</value>
            </property>
</configuration>




## 11) All setting for working YARN explainated in file: etc/hadoop/yarn-site.xml:
<configuration>
            <property>
                       <name>yarn.nodemanager.aux-services</name>
                       <value>mapreduce_shuffle</value>
            </property>
            <property>
                       <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                       <value>org.apache.hadoop.mapred.ShuffleHandler</value>
            </property>
            <property>
                       <name>yarn.resourcemanager.scheduler.address</name>
                       <value>master:8030</value>
            </property>
            <property>
                       <name>yarn.resourcemanager.address</name>
                       <value>master:8032</value>
            </property>
            <property>
                       <name>yarn.resourcemanager.webapp.address</name>
                       <value>master:8088</value>
            </property>
            <property>
                       <name>yarn.resourcemanager.resource-tracker.address</name>
                       <value>master:8031</value>
            </property>
            <property>
                       <name>yarn.resourcemanager.admin.address</name>
                       <value>master:8033</value>
            </property>
</configuration>


## 12) Again change Folder Permission for hduser:

sudo chown hduser:hadoop -R /usr/local/hadoop
create folder mkdir /usr/local/hadoop_store
sudo chown hduser:hadoop -R /usr/local/hadoop_store

sudo chmod -R 777 /usr/local/hadoop

sudo chmod -R 777 /usr/local/hadoop_store

## 13) Into bin folder:
##### cd /usr/local/hadoop/bin
##### ./hdfs namenode -format

##### cd /usr/local/hadoop/sbin
##### ./start-dfs.sh

## 14) If we have problems:
##### nslookup master   

##### netstat -tulpn   


## 15)
YARN on a Single Node
You can run a MapReduce job on YARN in a pseudo-distributed mode by setting a few parameters and running ResourceManager daemon and NodeManager daemon in addition.

The following instructions assume that 1. ~ 4. steps of the above instructions are already executed.

Configure parameters as follows:etc/hadoop/mapred-site.xml:

<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
etc/hadoop/yarn-site.xml:

<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
Start ResourceManager daemon and NodeManager daemon:

  $ sbin/start-yarn.sh
Browse the web interface for the ResourceManager; by default it is available at:

ResourceManager - http://localhost:8042/
Run a MapReduce job.

When you’re done, stop the daemons with:

  $ sbin/stop-yarn.sh
 


 
## 16) Additional steps for three nodes cluster

We copy our virtual mashine and made two additional copies:node1, node2(we must rename these names from master to node1 and node2),for one master and two nodes we must change ip address:
Nano  /etc/network/interfaces
master: 192.0.2.1
node1: 192.0.2.2
node2: 192.0.2.3
/etc/hostname

Before configuring the master and slave nodes, it’s important to understand the different components of a Hadoop cluster.

A master node keeps knowledge about the distributed file system, like the inode table on an ext3 filesystem, and schedules resources allocation. node-master will handle this role in this guide, and host two daemons:

The NameNode: manages the distributed file system and knows where stored data blocks inside the cluster are.
The ResourceManager: manages the YARN jobs and takes care of scheduling and executing processes on slave nodes.
Slave nodes store the actual data and provide processing power to run the jobs. They’ll be node1 and node2, and will host two daemons:

The DataNode manages the actual data physically stored on the node; it’s named, NameNode.
The NodeManager manages execution of tasks on the node.

Configure the System
Create Host File on Each Node
For each node to communicate with its names, edit the /etc/hosts file to add the IP address of the three servers. Don’t forget to replace the sample IP with your IP:

/etc/hosts:

10.0.2.20 master
10.0.2.21  node1
10.0.2.22 node2

Distribute Authentication Key-pairs for the Hadoop User
The master node will use an ssh-connection to connect to other nodes with key-pair authentication, to manage the cluster.

Login to node-master as the hadoop user, and generate an ssh-key:

ssh-keygen -t rsa -P ""

as we copied one main mashine into two it may say the certificates alredy exist in this case we dont need to copy certificates
Copy the key to the other nodes. It’s good practice to also copy the key to the master itself, so that you can also use it as a DataNode if needed. Type the following commands, and enter the hadoop user’s password when asked. If you are prompted whether or not to add the key to known hosts, enter yes:

##### ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@master
##### ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@node1
##### ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@node2

or alternative command: cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys

Configure Slaves
The file slaves is used by startup scripts to start required daemons on all nodes. Edit ~/hadoop/etc/hadoop/slaves to be:


node1
node2


Format HDFS

HDFS needs to be formatted like any classical file system. On node-master, run the following command:

hdfs namenode -format
Your Hadoop installation is now configured and ready to run.

Run and monitor HDFS
This section will walk through starting HDFS on NameNode and DataNodes, and monitoring that everything is properly working and interacting with HDFS data.

Start and Stop HDFS
Start the HDFS by running the following script from node-master:

start-dfs.sh
It’ll start NameNode and SecondaryNameNode on node-master, and DataNode on node1 and node2, according to the configuration in the slaves config file.

Check that every process is running with the jps command on each node. You should get on node-master (PID will be different):

21922 Jps
21603 NameNode
21787 SecondaryNameNode
and on node1 and node2:

19728 DataNode
19819 Jps
To stop HDFS on master and slave nodes, run the following command from node-master:

stop-dfs.sh

Attention: Before starting namenode, check network availability of nodes. If node1 and node2 not available in the netwrork then JPS on these nodes will schow just JPS service


