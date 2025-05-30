# **Hadoop 3.3.6 Multi Node Cluster Installation on Ubuntu**



## Jalankan command berikut
Download JDK 11, SSH, PDSH
```
sudo apt install openjdk-11-jdk ssh pdsh -y
```
## cek versi JDK
```
java -version
```

## Ubah hostname setiap komputernya
buka file hostname
```
sudo nano /etc/hostname
```
ubah isi file setiap komputernya sesuai role (master/slave123)

Contoh:

master: master

slave1: slave1

slave2: slave2

Setelah diubah setiap nama hostnamenya, reboot/restart PC nya

Bisa pake command
```
sudo reboot
```

## Cek IP duls
```
ip a
```
Contoh
192.168.1.10 di master maka ip di slavenya tinggal tambah 1 di ujung kanan nya

192.168.1.11 slave1

192.168.1.12 slave2

## Buka file hosts pake command ini (**HARUS ADA DI SEMUA KOMPUTER**)
```
sudo nano /etc/hosts
```
Masukin ip dan hostname nya, misal
```
192.168.1.10 master
192.168.1.11 slave1
192.168.1.12 slave2
```

## Masukin grup sudo
```
sudo usermod -aG sudo hduser
```

## Setup SSH
Generate SSH, run satu satu
```
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```
Copy SSH key ke semua node
```
ssh-copy-id hduser@master
ssh-copy-id hduser@slave1
ssh-copy-id hduser@slave2
```

## Install hadoop 3.3.6
```
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
```
Ekstrak hadoop
```
tar -xzvf hadoop-3.3.6.tar.gz
```
Ubah nama dan lokasi folder hadoop 
```
mv hadoop-3.3.6 ~/hadoop
```

### Set environment variable nya
Buka file .bashrc
```
sudo nano ~/.bashrc
```
Copas taro paling bawah file
```
export HADOOP_HOME=~/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
lalu run command
```
source ~/.bashrc
```

## Konfigurasi di file bernama hadoop-env.sh, core-site.xml, hdfs-site.xml, mapred-site.xml, yarn-site.xml

Buka file pake command ini, misal di file hadoop-env.sh
```
nano $HADOOP_CONF_DIR/hadoop-env.sh
```

Ini konfigurasi di setiap filenya

**File hadoop-env.sh**
```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```
**File core-site.xml**
```
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://master:9000</value>
  </property>
</configuration>
```
**File hdfs-site.xml**
```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///home/hduser/hadoopdata/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///home/hduser/hadoopdata/datanode</value>
  </property>
</configuration>
```
**File mapred-site.xml**
```
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```
**File yarn-site.xml**
```
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>master</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
```

### Konfigurasi workers
Buka file
```
nano $HADOOP_CONF_DIR/workers
```
copas semua
```
slave1
slave2
```

### Copy konfigurasi yang tadi ke setiap slave
Run command satu satu, disesuaikan dengan jumlah dan nama slavenya 
```
scp -r ~/hadoop hduser@slave1:~/
scp -r ~/hadoop hduser@slave2:~/
```

### Format Namenode
```
hdfs namenode -format
```

### Start hadoop nya
Run satu satu
```
start-dfs.sh
start-yarn.sh
```

### Cek web UI hadoop nya
pake hostname trus port nya

http://hostname:port

Misal

Namenode: ```http://master:9870```

ResourceManager: ```http://master:8088```

### Cara Stop hadoop 
Run satu satu
```
stop-dfs.sh
stop-yarn.sh
```

# Cara cek hadoop udah jalan di komputer lain
login ke user hadoop trus run jps
```
su - hadoop

jps
```
kalo dah jalan muncul
```
DataNode
NodeManager
```

# Cara upload dan lihat file dummy 200mb
Buat folder di HDFS
```
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/hadoop
```
Bikin file dummy 200mb
```
dd if=/dev/urandom of=sample_200mb.txt bs=1M count=200
```
Upload file ke HDFS
```
hdfs dfs -put sample_200mb.txt /user/hadoop/
```
Lihat apakah sudah terupload dan ada di folder
```
hdfs dfs -ls /user/hadoop/
```

