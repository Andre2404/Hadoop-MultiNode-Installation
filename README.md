# Hadoop-MultiNode-Installation

**Install SSH, PDSH, Java**
```
sudo install ssh
sudo install pdsh
sudo apt install openjdk-8-jdk
```

**Modif file .bashrc**
```
nano .bashrc
```
copy paste teks dan simpan paling bawah file
```
export PDSH_RCMD_TYPE=ssh
```
*note
ctrl + o trus enter = save file
ctrl + x = keluar file

**Konfigurasi SSH**
```
ssh-keygen -t rsa -P ""
```
enter enter aja

**Copy public key**
```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
pilih yes

**Verifikasi SSH**
```
ssh localhost
```

**Install Hadoop 3.3.6**
```
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
```

**Ekstrak Hadoop trus ganti nama**
```
tar -xvzf hadoop-3.3.6.tar.gz

mv hadoop-3.3.6 hadoop
```

**Konfigurasi file hadoop-env.sh**
```
nano ~/hadoop/etc/hadoop/hadoop-env.sh
```
copas trus taro paling bawah file
```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/
```

**Pindahin directory hadoop**
```
sudo mv hadoop /usr/local/hadoop
```

**Konfigurasi file environment**
```
sudo nano /etc/environment
```
hapus patsh trus copas path ini
```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/hadoop/bin:/usr/local/hadoop/sbin"

JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64/jre"
```

**Tambah user baru**
```
sudo adduser hadoopuser
```
trus run command dibawah ini satu satu
```
sudo usermod -aG hadoopuser hadoopuser
sudo chown hadoopuser:root -R /usr/local/hadoop/
sudo chmod g+rwx -R /usr/local/hadoop/
sudo adduser hadoopuser sudo
```

**Cek IP**
```
ip a
```
catet ip nya
misal :
master = 192.168.205.7
ip slave nya tinggal tambah 1 angka belakangnya
192.168.205.8
192.168.205.9

**Konfigurasi file hosts**
```
sudo nano /etc/hosts
```
masukin alamat IP master sama slavenya
contoh : 
192.168.205.7 hadoop-master
192.168.205.8 hadoop-slave1
192.168.205.9 hadoop-slave2

**Konfigurasi nama hostname**
```
sudo nano /etc/hostname
```
contoh
```
untuk master
hadoop-master

untuk slave
hadoop-slave1
hadoop-slave2
```

**Restart semua komputer setelah disave**
```
sudo reboot
```

**Selesai Restart ganti user ke hadoopuser**
```
su - hadoopuser
```

**Bikin ssh key baru di master aja**
```
ssh-keygen -t rsa
```

**Copy semua ssh key di master aja**
```
ssh-copy-id hadoopuser@hadoop-master
ssh-copy-id hadoopuser@hadoop-slave1
ssh-copy-id hadoopuser@hadoop-slave2
```
yes trus masukin password

**Konfigurasi core-site.xml di master aja**
```
sudo nano /usr/local/hadoop/etc/hadoop/core-site.xml
```
ganti configuration, copas
```
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://hadoop-master:9000</value>
</property>
</configuration>
```

**Konfigurasi file hdfs-site.xml di master aja**
```
sudo nano /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```
ganti configuration, copas
```
<configuration>
<property>
<name>dfs.namenode.name.dir</name><value>/usr/local/hadoop/data/nameNode</value>
</property>
<property>
<name>dfs.datanode.data.dir</name><value>/usr/local/hadoop/data/dataNode</value>
</property>
<property>
<name>dfs.replication</name>
<value>2</value>
</property>
</configuration>
```

**Konfigurasi worker/slave di master**
```
sudo nano /usr/local/hadoop/etc/hadoop/workers
```
ganti semuanya, copas
```
hadoop-slave1
hadoop-slave2
```

**Copy konfigurasi di hadoop master ke slave**
```
scp /usr/local/hadoop/etc/hadoop/* hadoop-slave1:/usr/local/hadoop/etc/hadoop/
scp /usr/local/hadoop/etc/hadoop/* hadoop-slave2:/usr/local/hadoop/etc/hadoop/
```

****
