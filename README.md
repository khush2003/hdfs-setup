## Initial Set Up
---
 Before we start we need to set a few things up. 

### Understanding
----
 - First of all one of the nodes (computers) must be a namenode, one secondary namenode and others are datanodes.
	 - Example: If you are 5 ppl in a group, you will have 1 namenode, 1 secondary namenode and 3 datanodes
	 - You can choose who will be the namenode and who will be the datanode or secondary name now. 


### Set up Hosts
----
 - Login into all nodes and get their IP address using this command on each node:
 - `hostname \-I | awk '{print $1}'`
 - Store these IP addresses in one file along with the hostname of the node. Hostname is of the format `s661csc371v0xx`
 - Now put this config in a file called hosts, use this code to access the file:
	 - `sudo vi /etc/hosts`
	 - Example contents of the file:
		 - ![Pasted image 20231116210636](https://github.com/khush2003/hdfs-setup/assets/98137724/9d3422b2-1c66-4248-8c5d-5a977a6ae1ad)

		 - Save the file by pressing `Shift` + `ZZ` on your keyboard or by typing `:wq`

> Please store the contents of /etc/hosts somewhere on your local machine as you will be needing to access this content later
### Disable Firewalls
---
Run this command on all nodes to disable firewalls:

```
sudo UFW disable
```


# Setting up Hadoop Cluster
---

1. Update Linux on **all nodes**

```
sudo apt-get update
```

2. On **all nodes** (all your devices) run this command

```
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
```

> This generates ssh key value pair for allowing nodes to access each other without passowrd


3. Now **only on** the node you decide to make your **namenode** (pick one node), run this command:
```
cat .ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

4. Now we will copy the ssh keys to all other nodes, run this code on **namenode only**:

```
scp .ssh/authorized_keys <hostname>:/home/sysadmin/.ssh/authorized_keys
```

> On this line of code replace `<hostname>` with the hostname of all other nodes including secondary namenode and datanodes

 Example:
 
```
scp .ssh/authorized_keys 10.4.85.114:/home/sysadmin/.ssh/authorized_keys
```

5. Install JDK on **all nodes**, run this command on all nodes:

```
sudo apt-get -y install openjdk-8-jdk-headless
```

To check if installed sucessfully:
```
java -version
```


6. Install hadoop on **all nodes** using these commands:
```
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.1.1/hadoop-3.1.1.tar.gz
```

```
tar -xzf hadoop-3.1.1.tar.gz
```

```
mv hadoop-3.1.1 hadoop
```

Make sure you have run all three commands in the correct order

7. Setting up environment variables on **all nodes**, open file using; `vi ~/.bashrc`

Add this code at the end of the file

```
export HADOOP_HOME=/home/sysadmin/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export YARN_HOME=${HADOOP_HOME}
```

Now run this code to reload the variables: `source ~/.bashrc`

8. Set up hadoop-env.sh on **all nodes** by opening the file using: `vi ~/hadoop/etc/hadoop/hadoop-env.sh`

Append this at the end of the file (does not nessarily have to be end of file you can also add it in any blank space)
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

Example:      ![Pasted image 20231116212359](https://github.com/khush2003/hdfs-setup/assets/98137724/2783e017-2c2c-4fb4-a3db-45c6c1602a60)


9. Update core-site.xml on **all nodes** by opening the file using: `vi ~/hadoop/etc/hadoop/core-site.xml`

Change `<configuration> </configuration>` to 

```
<configuration> 
	<property> 
		<name>fs.defaultFS</name> 
		<value>hdfs://<namenode ip>:8020</value> 
	</property> 
</configuration>
```

Replace `<namenode ip>` with the ip of your name node you picked. Eg, our group picked `10.4.85.110` as the namenode ip


10. Update hdfs-site.xml on **all nodes** by opening the file using: `vi ~/hadoop/etc/hadoop/hdfs-site.xml` 


Change `<configuration> </configuration>` to 

```
<configuration> 
	<property> 
		<name>dfs.replication</name> 
		<value>3</value> 
	</property> 
	<property> 
		<name>dfs.namenode.name.dir</name> 
		<value>file:///usr/local/hadoop/hdfs/data</value> 
	</property> 
	<property> 
		<name>dfs.datanode.data.dir</name> 
		<value>file:///usr/local/hadoop/hdfs/data</value> 
	</property> 
	<property>
    		<name>dfs.secondary.http.address</name>
    		<value><secondary namenode ip>:50090</value>
	</property>
</configuration>

```

Where you 
- change the value of replication to the number of data nodes in your cluster
- change `<secondary namenode ip>` to your secondary name  node’s ip

11. Update yarn-site.xml on **all nodes** by opening the file using: `vi ~/hadoop/etc/hadoop/yarn-site.xml` 

Change `<configuration> </configuration>` to 

```
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
		<name>yarn.resourcemanager.hostname</name> 
		<value><namenode ip></value> 
	</property> 
</configuration>
```

Replace `<namenode ip>` with the ip of your name node you picked. 

12. Update mapred-site.xml on **all nodes** by opening the file using: `vi ~/hadoop/etc/hadoop/mapred-site.xml` 

Change `<configuration> </configuration>` to 
```
<configuration> 
	<property> 
		<name>mapreduce.jobtracker.address</name> 
		<value><namenode ip>:54311</value> 
	</property> 
	<property> 
		<name>mapreduce.framework.name</name> 
		<value>yarn</value> 
	</property>
  <property>
          <name>yarn.app.mapreduce.am.env</name>
          <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
  </property>
  <property>
          <name>mapreduce.map.env</name>
          <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
  </property>
  <property>
          <name>mapreduce.reduce.env</name>
          <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
  </property>
</configuration>
```

Replace `<namenode ip>` with the ip of your name node you picked. 

13. Create a data folder on **all nodes**

```
sudo mkdir -p /usr/local/hadoop/hdfs/data
```

```
sudo chown sysadmin:sysadmin -R /usr/local/hadoop/hdfs/data
```

```
chmod 700 /usr/local/hadoop/hdfs/data
```

14. Create master file on **all nodes** using the command, `vi ~/hadoop/etc/hadoop/master`
 Change the contents of the file to only have ip addresses of namenode and secondary name node. Example:

 ![Pasted image 20231116214317](https://github.com/khush2003/hdfs-setup/assets/98137724/12d0794c-c58f-4df8-aeba-b60b796b3704)

 
15. Create workers file on **all nodes** using the command, `vi ~/hadoop/etc/hadoop/workers`
 Change the contents of the file to only have ip addresses of all datanodes node. Example:

![Pasted image 20231116214456](https://github.com/khush2003/hdfs-setup/assets/98137724/57453c92-cb98-4308-a655-ca847dba4e6b)

16. Now only on name node run this code to format it

```
hdfs namenode -format
```

17. You can start the hadoop cluster using the following command

```
start-dfs.sh
```

Sample output:
![Pasted image 20231116214816](https://github.com/khush2003/hdfs-setup/assets/98137724/00663df2-261c-4e11-9619-eeccf4516957)

 
 You can check running hdfs using the following commmands:
 - `jps`: Lists the running services, on namenode it will list jps and namenode, on secondary name node it will list jps and secondary name node, so on and so forth.
 - `hdfs dfsadmin -report`: This will show you a log to see what is running and all the configuration of the nodes. If it does not list anything or lists something but with only 0s on all fields then something went wrong above please make sure you have followed all steps properly. 

You can also check the status of the hdfs by visiting this url on your local computer: `http://namenode_ip:9870 ` where namenode_ip is the ip of your namenode

18. You can create a new home directory on the hdfs server using the following command

```
hdfs dfs -mkdir -p /user/sysadmin/
```

You can also create another directory for storing the books you might want to download using:

```
hdfs dfs -mkdir books
```

If you run both commands above you will get a directory on your hdfs as follows:

`/user/sysadmin/books` Keep in mind you will be at the home directory(`/user/sysadmin/`) by default. 

19. Get books from gutenberg project using

```
wget -0 <filename.txt> <url of .txt file>
```

Example: 

```
wget -O alice.txt https://www.gutenberg.org/files/11/11-0.txt
```

> This will save the file on the node not on hdfs, so you need to upload it to hdfs


19. Upload the file to the hdfs server using the following command:

```
hdfs dfs -put <location of file on node> <location on hdfs>
```

Example:

```
hdfs dfs -put alice.txt books/alice.txt
```

You can list file using the following command:

```
hdfs dfs -ls
```

or 

```
hdfs dfs -ls books/
```

> [!IMPORTANT]
> If you get en error saying 0 datanodes running when uploading file to hdfs contact me

20. You can stop hdfs using:

```
stop-dfs.sh
```

> [!NOTE]
> When running map-reduce task
> When you will be running map reduce task, it is not enough to run `start-dfs.sh` instead you should use `start-all.sh` this will also start node manager and resource manager which are required for hdfs.
> 
> To stop from `start-all.sh` you can run `stop-all.sh`


> [!NOTE]
> Code to run map-reduce when you have a jar file already (I have not listed the steps to make a jar file for wordcount map-reduce task you have to do that yourself)
> 
> ```
> hadoop jar <location of jar file> <java class to run> <location of input file on hdfs> <location of the output folder>
> ```
>  Keep in mind that `<location of input file on hdfs>` is the absolute path to that file, in the above example it would be `/user/sysadmin/books/alice.txt` 
>  
> The output directory that is also absolute path example `/user/sysadmin/output/`, you do not need to mkdir output, it will automatically be created.



