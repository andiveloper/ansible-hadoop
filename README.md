# ansible-hadoop
[![Build Status](https://travis-ci.org/andiveloper/ansible-hadoop.svg?branch=master)](https://travis-ci.org/andiveloper/ansible-hadoop)

An Ansible role which can be used to create a multi-node Apache Hadoop cluster for development purposes.

# Features
- Deploy Datanodes
- Deploy NameNodes (single or HA)
- Deploy Manager Nodes (ResourceManager, NodeManager, HistoryServer)
- Offline or online install
- Download sample JAR which can be used to execute teragen, terasort jobs
- Default install dir: `/opt/hadoop`

# Sample Usage
The role relies on specific group names. A sample inventory file could look like this:

```
# Sample inventory.ini for NameNode HA
[hadoop_namenodes]
node-1

[hadoop_datanodes]
node-1
node-2

[hadoop_managers] 
# these hosts will run: ResourceManager, NodeManager and MapReduce Job History server
node-1

[hadoop:children]
hadoop_namenodes
hadoop_datanodes
hadoop_managers

[hadoop:vars]
# If set to True all existing data (if any) will be deleted
hadoop_reformat_namenode=True

# Shared Edits Dir is needed for HA if multiple NameNodes are specified, so use only if more than one NameNode is specified
# This directory must be a shared directory and accessible from all the NameNodes 
# (e.g. you have to set up a NFS share and mount it on all the NameNodes)
# hadoop_hdfs_site={"dfs.namenode.shared.edits.dir": "/mnt/gpfs0/hdfs-ha"}

# Online install, each node needs internet access
hadoop_distro_url=http://mirrors.ukfast.co.uk/sites/ftp.apache.org/hadoop/common/hadoop-3.1.2/hadoop-3.1.2.tar.gz
hadoop_jre_url=https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u222-b10/OpenJDK8U-jre_x64_linux_hotspot_8u222b10.tar.gz
hadoop_sample_jar_urls=['https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-mapreduce-examples/3.1.2/hadoop-mapreduce-examples-3.1.2.jar']

# For offline install download JRE and Hadoop distro manually, place them on the ansible host and uncomment the following two vars
#hadoop_distro_local_pkg=/root/hadoop-3.1.2.tar.gz
#hadoop_jre_local_pkg=/root/OpenJDK8U-jre_x64_linux_hotspot_8u222b10.tar.gz
#hadoop_sample_jar_pkgs=['/root/hadoop-mapreduce-examples-3.1.2.jar']
```

```
# Sample playbook
---
- hosts: hadoop
  remote_user: root
  gather_facts: no
  roles:
    - hadoop
```

## Testing your cluster setup
If the playbook finished successfully you can run a sample job using the downloaded sample JAR, for example:
```
su hdfs
export PATH=$PATH:/opt/hadoop/current/bin
OUTPUT_DIR=/performance2

hdfs dfs -rm -r -f $OUTPUT_DIR

time yarn jar /opt/hadoop/samples/hadoop-mapreduce-examples-3.1.2.jar teragen -Dmapreduce.jobs.maps=128 -Ddfs.blocksize=128M 10000000 $OUTPUT_DIR/teragen_1G_128maps

time yarn jar /opt/hadoop/samples/hadoop-mapreduce-examples-3.1.2.jar terasort   -D mapreduce.map.output.compress=true  $OUTPUT_DIR/teragen_1G_128maps $OUTPUT_DIR/teragen_1G_128maps_sorted_1

```
