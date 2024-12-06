# Hadoop-Ecosystem
Here is the complete configuration guide for Hadoop, Flume, Hive, and Sqoop in one file:

---

### **Hadoop, Flume, Hive, and Sqoop Setup and Configuration Guide**

---

### **1. Single Node Hadoop Cluster Setup**

#### **Step 1: Install Hadoop**

1. **Update System**
   ```bash
   sudo apt-get update
   sudo apt-get upgrade
   ```

2. **Install Java**
   Hadoop requires Java. To install it:
   ```bash
   sudo apt-get install openjdk-8-jdk
   ```

3. **Download Hadoop**
   Download the latest stable version of Hadoop:
   ```bash
   wget https://archive.apache.org/dist/hadoop/core/hadoop-3.3.0/hadoop-3.3.0.tar.gz
   ```

4. **Extract and Move Hadoop**
   Extract the Hadoop tar file and move it to the `/usr/local` directory:
   ```bash
   tar -xzvf hadoop-3.3.0.tar.gz
   sudo mv hadoop-3.3.0 /usr/local/hadoop
   ```

5. **Set Hadoop Environment Variables**
   Add the following lines to `~/.bashrc`:
   ```bash
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
   export HADOOP_HOME=/usr/local/hadoop
   export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
   export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   export HADOOP_MAPRED_HOME=$HADOOP_HOME
   export HADOOP_COMMON_HOME=$HADOOP_HOME
   export HADOOP_HDFS_HOME=$HADOOP_HOME
   ```

6. **Apply the Changes**
   Run the following command to apply the changes:
   ```bash
   source ~/.bashrc
   ```

#### **Step 2: SSH Setup**

To allow Hadoop to run without entering a password:
1. **Generate SSH Keys**
   ```bash
   ssh-keygen -t rsa
   ```

2. **Copy SSH Key**
   Copy the public key to `authorized_keys`:
   ```bash
   cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
   ```

3. **Test SSH**
   Test if the machine can SSH into itself without a password:
   ```bash
   ssh localhost
   ```

#### **Step 3: Hadoop Configuration**

1. **Edit `core-site.xml`**
   Add the following configurations in `/usr/local/hadoop/etc/hadoop/core-site.xml`:
   ```xml
   <configuration>
       <property>
           <name>fs.defaultFS</name>
           <value>hdfs://localhost:9000</value>
       </property>
   </configuration>
   ```

2. **Edit `hdfs-site.xml`**
   Configure HDFS in `/usr/local/hadoop/etc/hadoop/hdfs-site.xml`:
   ```xml
   <configuration>
       <property>
           <name>dfs.replication</name>
           <value>1</value>
       </property>
       <property>
           <name>dfs.namenode.name.dir</name>
           <value>file:///usr/local/hadoop/hadoop_data/name</value>
       </property>
       <property>
           <name>dfs.datanode.data.dir</name>
           <value>file:///usr/local/hadoop/hadoop_data/data</value>
       </property>
   </configuration>
   ```

3. **Edit `mapred-site.xml`**
   Configure MapReduce in `/usr/local/hadoop/etc/hadoop/mapred-site.xml`:
   ```xml
   <configuration>
       <property>
           <name>mapreduce.framework.name</name>
           <value>yarn</value>
       </property>
   </configuration>
   ```

4. **Edit `yarn-site.xml`**
   Configure YARN in `/usr/local/hadoop/etc/hadoop/yarn-site.xml`:
   ```xml
   <configuration>
       <property>
           <name>yarn.resourcemanager.address</name>
           <value>localhost:8032</value>
       </property>
       <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
       </property>
   </configuration>
   ```

#### **Step 4: Format the NameNode**

```bash
hdfs namenode -format
```

#### **Step 5: Start Hadoop Services**

Start the HDFS and YARN services:
```bash
start-dfs.sh
start-yarn.sh
```

#### **Step 6: Verify Hadoop Installation**

Check the status of Hadoop services:
```bash
jps
```

You should see `NameNode`, `DataNode`, `ResourceManager`, and `NodeManager` running.

---

### **2. Apache Flume (Data Integration) Setup**

#### **Step 1: Install Flume**

1. **Download and Install Flume**
   Download the latest stable version of Flume:
   ```bash
   wget https://archive.apache.org/dist/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz
   tar -xzvf apache-flume-1.9.0-bin.tar.gz
   sudo mv apache-flume-1.9.0-bin /usr/local/flume
   ```

#### **Step 2: Configure Flume**

1. **Create Flume Configuration File**
   Create a file called `flume.conf` in the `/usr/local/flume/conf/` directory:
   ```properties
   # Source
   agent.sources = r1
   agent.sources.r1.type = netcat
   agent.sources.r1.bind = localhost
   agent.sources.r1.port = 44444

   # Sink
   agent.sinks = k1
   agent.sinks.k1.type = hdfs
   agent.sinks.k1.hdfs.path = hdfs://localhost:9000/flume/logs
   agent.sinks.k1.hdfs.fileType = DataStream
   agent.sinks.k1.hdfs.writeFormat = Text

   # Channel
   agent.channels = c1
   agent.sources.r1.channels = c1
   agent.sinks.k1.channel = c1
   agent.channels.c1.type = memory
   agent.channels.c1.capacity = 10000
   agent.channels.c1.transactionCapacity = 100
   ```

2. **Start Flume Agent**
   Start the Flume agent using the configuration file:
   ```bash
   /usr/local/flume/bin/flume-ng agent --conf /usr/local/flume/conf --conf-file /usr/local/flume/conf/flume.conf --name agent
   ```

---

### **3. Apache Hive (Data Warehousing) Setup**

#### **Step 1: Install Hive**

1. **Download and Install Hive**
   ```bash
   wget https://archive.apache.org/dist/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
   tar -xzvf apache-hive-3.1.2-bin.tar.gz
   sudo mv apache-hive-3.1.2-bin /usr/local/hive
   ```

2. **Set Hive Environment Variables**
   Add the following lines to `~/.bashrc`:
   ```bash
   export HIVE_HOME=/usr/local/hive
   export PATH=$PATH:$HIVE_HOME/bin
   ```

#### **Step 2: Configure Hive**

1. **Edit `hive-site.xml`**
   Create and configure the `hive-site.xml` file in `/usr/local/hive/conf/`:
   ```xml
   <configuration>
       <property>
           <name>hive.metastore.uris</name>
           <value>thrift://localhost:9083</value>
       </property>
       <property>
           <name>hive.metastore.warehouse.dir</name>
           <value>/user/hive/warehouse</value>
       </property>
   </configuration>
   ```

2. **Initialize the Metastore**
   ```bash
   schematool -initSchema -dbType derby
   ```

3. **Start Hive**
   ```bash
   hive
   ```

---

### **4. Apache Sqoop (Data Import/Export) Setup**

#### **Step 1: Install Sqoop**

1. **Download and Install Sqoop**
   ```bash
   wget https://archive.apache.org/dist/sqoop/1.4.7/sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
   tar -xzvf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
   sudo mv sqoop-1.4.7.bin__hadoop-2.6.0 /usr/local/sqoop
   ```

2. **Set Sqoop Environment Variables**
   Add the following lines to `~/.bashrc`:
   ```bash
   export SQOOP_HOME=/usr/local/sqoop
   export PATH=$PATH:$SQOOP_HOME/bin
   ```

3. **Configure Sqoop for Hadoop**
   Edit the `/usr/local/sqoop/conf/sqoop-env.sh` file to include the correct `HADOOP_HOME` and `HIVE_HOME`.

#### **Step 2: Test Sqoop**

To test Sqoop, run the following command:
```bash
sqoop version
```



### Author : Rushikesh Shinde
### Contact : +91 9623548002
