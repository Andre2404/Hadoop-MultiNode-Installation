# **Hadoop 3.3.6 Multi Node Cluster Installation on Ubuntu**

Buat yang pake VM (VirtualBox) setting network VM nya begini
![Network Settings](https://github.com/Imam-Riyanto090/Hadoop-MultiNode-Installation/blob/main/SettingVM.jpg)

kalo gaada perintah **Jalankan di semua komputer** berarti cuma di master aja

## Update dan Download JDK 11, SSH, PDSH (**Jalankan di semua komputer**)
```
sudo apt update && sudo apt upgrade -y
sudo apt install openjdk-11-jdk ssh pdsh -y
```

## Ubah hostname setiap komputernya (**Jalankan di semua komputer**)
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

## Setup SSH Key 
run satu satu
```
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
copy key ke semua slave
```
ssh-copy-id hduser@master
ssh-copy-id hduser@slave1
ssh-copy-id hduser@slave2
```

## Konfigurasi File hosts
cek IP dulss di **semua komputer**
```
ip a
```
Catet IP nya, buka file hosts pake command ini di master aja
```
sudo nano /etc/hosts
```
Masukin ip dan hostname nya, misal
```
192.168.1.10 master
192.168.1.11 slave1
192.168.1.12 slave2
```

## Salin konfigurasi hosts ke komputer slave
karena butuh akses sudo maka nama file hosts nya kita ganti dulu ke hosts_temp
```
scp /etc/hosts hduser@slave1:~/hosts_temp
scp /etc/hosts hduser@slave2:~/hosts_temp
```
Kemudian login ke setiap komputer slave dan copy ke lokasi /etc/hosts dengan sudo
login ke setiap slave pake command
```
ssh slave1
ssh slave2
```
trus ganti namanya pake command ini di setiap slave
```
sudo mv ~/hosts_temp /etc/hosts
```
kalo udah logout pake command
```
exit
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

## Salin konfigurasi .bashrc ke komputer slave
```
scp ~/.bashrc hduser@slave1:~/
scp ~/.bashrc hduser@slave2:~/
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

## Konfigurasi workers
Buka file
```
nano $HADOOP_CONF_DIR/workers
```
copas semua
```
slave1
slave2
```

## Copy konfigurasi yang tadi ke setiap slave
Run command satu satu, disesuaikan dengan jumlah dan nama slavenya 
```
scp -r ~/hadoop hduser@slave1:~/
scp -r ~/hadoop hduser@slave2:~/
```

## Format Namenode
```
hdfs namenode -format
```

## Start hadoop nya
Run satu satu
```
start-dfs.sh
start-yarn.sh
```

## Cek web UI hadoop nya
pake hostname trus port nya

http://hostname:port

Misal

Namenode: ```http://master:9870```

ResourceManager: ```http://master:8088```

## Cara Stop hadoop 
Run satu satu
```
stop-dfs.sh
stop-yarn.sh
```

## Cara cek hadoop udah jalan di komputer lain
login ke user hadoop trus run jps
```
jps
```
kalo dah jalan muncul
```
DataNode
NodeManager
```

## Cara upload, download dan lihat file di HDFS
Buat folder di HDFS
```
hdfs dfs -mkdir /user/hduser/input
```
Bikin file dummy 200mb
```
dd if=/dev/urandom of=sample_200mb.txt bs=1M count=200
```
Upload file ke HDFS
```
hdfs dfs -put sample_200mb.txt /user/hduser/input/
```
Lihat apakah sudah terupload dan ada di folder
```
hdfs dfs -ls /user/hduser/input/
```
lihat isi file
```
hdfs dfs -cat /user/hduser/input/sample_200mb.txt
```
Download file ke lokal
```
hdfs dfs -get /user/hduser/input/sample_200mb.txt
```
