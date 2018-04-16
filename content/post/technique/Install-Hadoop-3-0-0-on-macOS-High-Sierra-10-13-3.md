---
title: "Install Hadoop 3.0.0 on macOS High Sierra 10.13.3"
date: 2018-03-15T15:33:48-05:00
lastmod: 2018-04-14T22:31:00-05:00

tags: ["Big Data", "Hadoop"]
categories: ["Technique"]


---

{{% figure class="center" src="/img/hadoop-logo.png" alt="hadoop-logo" title="" %}}

## 1. Check if Java is installed.
In terminal, input:
``` bash
$ java -version
```
output:
<!--more-->
``` bash
java version "1.8.0_66"
Java(TM) SE Runtime Environment (build 1.8.0_66-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.66-b17, mixed mode)
```
If Java is not installed, download Java from `https://java.com/en/download/` and install it first.

<!-- more -->

## 2. Download Hadoop.
We use the stable version 3.0.0.
`http://apache.claz.org/hadoop/common/hadoop-3.0.0/hadoop-3.0.0.tar.gz`

## 3. Configure macOS environment.
In terminal, input:
```bash
$ ssh localhost
```
If you can login localhost successfully after providing password, you are good to go to the next step. If not, go to System Preferences > Sharing, check Remote Login, allow the user who is going to use hadoop to remote login. Or in terminal, input:
```bash
$ ssh-keygen -t rsa 
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
The authorization of each file and folder have to be correct to make passwordless login work correctly.
```bash
$ chmod 600 ~/.ssh/authorized_keys
$ chmod 700 ~/.ssh
$ chmod 755 <the parent folder of .ssh>
```

## 4. Setup environment variables
In terminal:
```bash
$ sudo vim ~/.bash_profile
```
Add following lines into `.bash_profile` file: (Please change the path according to your system)
```bash
# Java Path
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home
export JRE_HOME=$JAVA_HOME/jre

# Hadoop Path
export HADOOP_HOME=/Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export HADOOP_HDFS_HOME=${HADOOP_HOME}

# Add Java Path to $PATH.
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin: $PATH

# Add Hadoop Path to $PATH
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```
After adding the path to `$PATH` variable, you don’t have to go to each directory to execute the command, and when you run bash script, you can just type `script_name.sh` instead of `source script_name.sh` or `. script_name.sh`.

Save file and execute the following command for the changes to take effect: 
```bash
$ source ~/.bash_profile
```
Check in terminal, input: 
```bash
$ hadoop version
```
output:
```bash
Hadoop 3.0.0
Source code repository https://git-wip-us.apache.org/repos/asf/hadoop.git -r c25427ceca461ee979d30edd7a4b0f50718e6533
Compiled by andrew on 2017-12-08T19:16Z
Compiled with protoc 2.5.0
From source with checksum 397832cb5529187dc8cd74ad54ff22
This command was run using /Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/share/hadoop/common/hadoop-common-3.0.0.jar
```

## 5. Configure files in hadoop folder
The configuration files are in `/Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/etc/hadoop/`
### 1) hadoop-env.sh
Let the `~/.bash_profile` to determine the Java path. Add:
```bash
$ export JAVA_HOME=${JAVA_HOME}
```
Let the `~/.bash_profile` to determine the Hadoop path. Add:
```bash
$ export HADOOP_HOME=${HADOOP_HOME}
```
### 2) core-site.xml
Add the following lines between `<configuration>`  and  `</configuration>`:
```xml
    <property>  
        <name>fs.defaultFS</name>  
        <value>hdfs://localhost:9000</value>
    </property>
```
### 3) hdfs-site.xml
Change the content to the following lines between `<configuration>` and `</configuration>`:
```xml
    <property>  
       <name>dfs.replication</name>  
       <value>1</value>  
    </property>
```
### 4) mapred-site.xml
Copy `mapred-site.xml.template` and rename to `mapred-site.xml`.
Add the following lines between `<configuration>` and `</configuration>`:
```xml
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    
    <property>
        <name>mapreduce.admin.user.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_COMMON_HOME</value>
    </property>
    
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_COMMON_HOME</value>
    </property>
```
### 5) yarn-site.xml
Add the following lines between `<configuration>` and `</configuration>`:
```xml
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
```

## 6. Format
Go to `/Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/bin` (In fact you don’t have to, since you have add this path to `$PATH`), and execute:
```bash
$ hadoop namenode -format
```

## 7. Start Hadoop
Go to `/Users/jz0006/GoogleDrive/Hadoop/hadoop-2.9.0/sbin` (In fact you don’t have to, since you have add this path to `$PATH`),

Execute:
```bash
$ . start-dfs.sh # or just type "start-dfs.sh"
```
This script will start namenode, datanode, and secondary namenodes on `[localhost]`.

Execute:
```bash
$ . start-yarn.sh # or just type "start-yarn.sh"
```
This script will start resourcemanager and nodemanager.

## 8. Check if Hadoop works well
Type `http://localhost:9870` and `http://localhost:8088` in browser to check if the webpages can be loaded successfully.

## ISSUES:
1. When start SecondaryNameNode, program took hostname as localhost to connect node, which causes connection denied. I have to set hostname to `‘localhost’` to cheat the program to work around:

```bash
$ sudo scutil --set HostName localhost
```
I think there should be somewhere to specify the address of `SecondaryNameNode`.

2. In `hdfs-site.xml`, I tried to set permanent path to data node and name node by creating directory folder and adding the following lines to `hdfs-site.xml` file:

```xml
    <property>
        <name>dfs.name.dir</name>
        <value>file:///Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/hadoopinfra/hdfs/namenode </value>
    </property>
    
    <property>
        <name>dfs.data.dir</name> 
        <value>file:///Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/hadoopinfra/hdfs/datanode </value> 
    </property>
```
But with this setup, the name node cannot start. So I delete these lines and use the default `/tmp` path.




