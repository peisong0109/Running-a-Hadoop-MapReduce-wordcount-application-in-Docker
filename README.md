# Running-a-Hadoop-MapReduce-wordcount-application-in-Docker
# Running a MapReduce wordcount application in Docker

##### Background
One simple application is to count the occurrences of certain keywords related to SDGs such as “green,” “health,” “education,” “equality,” etc. With a traditional approach, which does not involve the concept of distributed processing, the job of word counting might not be easy or quick. In addition, it would be very computationally expensive.

##### Intention
The purpose of this project is to develop a simple word count application that demonstrates the working principle of MapReduce, involving multiple Docker Containers as the clients, to meet the requirements of distributed processing, using Python SDK for Docker.


---

## Configure and save the hadoop image

##### Step 1:
Pull the ubuntu image from docker:

```
docker pull ubuntu
```
![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step1.png)

Create the container based on the ubuntu image:

```
docker run -it --name myhadoop -p 80:80 ubuntu

# docker run: Create and run a container.
# -it: Interactive model.
# --name: Assign a name to the container.
# -p: Publish a container's port to the host.
# ubuntu: The name of the image.
```


##### Step 2:
Install the java:

```
apt update
apt install openjdk-8-jdk
java -version
#Verify the java has been installed.
update-alternatives --config java
#Check the path of the java.
```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step2.png)

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step2-2.png)

##### Step 3:
Install the required tools, including vim, config, ssh:

```
apt install vim
apt install net-tools
apt-get install openssh-server openssh-client
cd
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
echo "service ssh start" >> ~/.bashrc

```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step3-1.png)

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step3-2.png)

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step3-3.png)

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step3-4.png)


##### Step 4:
Download hadoop:

```
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz
tar -zxvf hadoop-3.2.1.tar.gz -C /usr/local/
cd /usr/local/
mv hadoop-3.2.1 hadoop 
```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step4-1.png)

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step4-2.png)

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step4-3.png)


##### Step 5:
Modify the environment variables:

```
vim /etc/profile
```
Add the following variables:

```
#java
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64	
#Your java path
export JRE_HOME=${JAVA_HOME}/jre    
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib    
export PATH=${JAVA_HOME}/bin:$PATH
#hadoop
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_COMMON_HOME=$HADOOP_HOME 
export HADOOP_HDFS_HOME=$HADOOP_HOME 
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME 
export HADOOP_INSTALL=$HADOOP_HOME 
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native 
export HADOOP_CONF_DIR=$HADOOP_HOME 
export HADOOP_LIBEXEC_DIR=$HADOOP_HOME/libexec 
export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native:$JAVA_LIBRARY_PATH
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HDFS_DATANODE_USER=root
export HDFS_DATANODE_SECURE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export HDFS_NAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step5-1.png)

Run:

```
echo "source /etc/profile" >> ~/.bashrc
```

##### Step 6:
Modify the config files of hadoop:

```
#Get into the hadoop config file:
cd $HADOOP_CONF_DIR
```

###### Add the following variables into hadoop-env.sh:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step6-1.png)


###### Add the following variables into core-site.xml:

```
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://h01:9002</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoop/tmp</value>
    </property>
</configuration>

#You must make sure that the port 9002 is not occupied.
```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step6-2.png)

###### Create new folders:

```
mkdir -p /data/hadoop/tmp
mkdir -p /data/hadoop/hdfs
mkdir /data/hadoop/hdfs/name
mkdir /data/hadoop/hdfs/data

```

###### Add the following variables into hdfs-site.xml:

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/data/hadoop/hdfs/name</value>
    </property>
    <property>
        <name>dfs.namenode.data.dir</name>
        <value>/data/hadoop/hdfs/data</value>
    </property>
</configuration>

```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step6-3.png)

###### Add the following variables into mapred-site.xml:

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>
            /usr/local/hadoop/etc/hadoop,
            /usr/local/hadoop/share/hadoop/common/*,
            /usr/local/hadoop/share/hadoop/common/lib/*,
            /usr/local/hadoop/share/hadoop/hdfs/*,
            /usr/local/hadoop/share/hadoop/hdfs/lib/*,
            /usr/local/hadoop/share/hadoop/mapreduce/*,
            /usr/local/hadoop/share/hadoop/mapreduce/lib/*,
            /usr/local/hadoop/share/hadoop/yarn/*,
            /usr/local/hadoop/share/hadoop/yarn/lib/*
        </value>
    </property>
</configuration>

```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step6-4.png)

###### Add the following variables into yarn-site.xml:

```
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>h01</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>

```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step6-5.png)

##### Step 7:
Save the configured container into a docker image:

```
docker ps -a  
# Find the id of the current container.
docker commit -m "install haddop" 8a2a24b54e6e ubuntu:hadoop
# 8a2a24b54e6e:container id
# ubuntu:hadoop：The name of your new-built image.

```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step7.png)


---
## Run hadoop cluster within docker containers

##### Step 1:
Open three terminals and create one container in each terminal:

```
#Start three containers
docker run -it -h h01 --name h01 -p 9870:9870 -p 8088:8088 -p 9002:9002 ubuntu:hadoop /bin/bash
docker run -it -h h02 --name h02 ubuntu:hadoop /bin/bash
docker run -it -h h03 --name h03 ubuntu:hadoop /bin/bash
# -h：Container's hostname

```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step2.1-1.png)

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step2.1-2.png)

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step2.1-3.png)

##### Step 2:
Check the IP of the current container.

```
cat /etc/hosts
```
Modify the hosts and save the IP of each container.

```
vim /etc/hosts
```

![image](https://github.com/peisong0109/Running-a-Hadoop-MapReduce-wordcount-application-in-Docker/blob/main/screenshots/step2.2-1.png)

Verify that the h01 can connect to the h02 and h03 well.

```
ssh h01
ssh h02
```
###### Modify the workers:

Delete localhost and input all the three hostname

```
cd $HADOOP_CONF_DIR
vim workers
```

##### Step 3:
Start the hadoop cluster

```
#Format the namecode THE FIRST TIME you start the cluster (Only executed once!)
hdfs namenode -format
#Start the cluster
start-all.sh

```

##### Step 4:
Check the state of each node.

```
root@h01:~# jps
```

---
## Test the hadoop with the WordCount application

##### Step 1:
We choose the licence.txt as the file used for counting the words.

```
root@h01:~# cat $HADOOP_HOME/LICENSE.txt > file.txt
```

##### Step 2:
Input the file into the hadoop:

```
hadoop fs -ls /
hadoop fs -mkdir /input
hadoop fs -put file.txt /input
hadoop fs -ls /input

```

##### Step 3:
Run the WordCount application:

```
root@h01:~# hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.1.jar wordcount /input /output
```

##### Step 4:
Show the counting results:

```
root@h01:~# hadoop fs -ls /output
Found 2 items
-rw-r--r--   2 root supergroup          0 2020-04-07 12:34 /output/_SUCCESS
-rw-r--r--   2 root supergroup      35324 2020-04-07 12:34 /output/part-r-00000
root@h01:~# hadoop fs -cat /output/part-r-00000

```


