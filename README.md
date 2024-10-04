# Hadoop Cluster Setup on Virtual Machine using Multipass
---
# Quick start
```bash
brew install --cask multipass
multipass launch jammy --name hadoop-vm --disk 6G

multipass shell hadoop-vm
cd ~ && git clone https://github.com/bdbao/Hadoop-VM
sudo apt update
sudo apt install openjdk-8-jdk-headless -y
java -version
sudo sh -c 'echo export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-arm64" >> /etc/environment'
source /etc/environment

# cat <<EOF >> ~/.bashrc: This begins appending a block of text to the end of ~/.bashrc
cat <<EOF >> ~/.bashrc
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-arm64
export PATH=\$PATH:/usr/lib/jvm/java-8-openjdk-arm64/bin
export HADOOP_HOME=~/hadoop-3.2.3/
export PATH=\$PATH:\$HADOOP_HOME/bin
export PATH=\$PATH:\$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=\$HADOOP_HOME
export YARN_HOME=\$HADOOP_HOME
export HADOOP_CONF_DIR=\$HADOOP_HOME/etc/hadoop
export HADOOP_COMMON_LIB_NATIVE_DIR=\$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=\$HADOOP_HOME/lib/native"
export HADOOP_STREAMING=\$HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.2.3.jar
export HADOOP_LOG_DIR=\$HADOOP_HOME/logs
export PDSH_RCMD_TYPE=ssh
EOF

source ~/.bashrc
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.2.3/hadoop-3.2.3.tar.gz
tar -xzvf hadoop-3.2.3.tar.gz
cd hadoop-3.2.3
cp -r ~/Hadoop-VM/hadoop-3.2.3/etc .
# Setup SSH for Hadoop (Ubuntu) User
chmod +x ~/* # (Optional)
hdfs namenode -format
start-all.sh
jps
ip addr show # get instance-ip
# Go to: : http://<instance-ip>:9870 (HDFS), http://<instance-ip>:8088 (YARN)
stop-all.sh
```

# Build from scratch
## Create Multipass instance (Ubuntu machine)
```bash
mkdir ~/Hadoop-VM && cd ~/Hadoop-VM
brew install --cask multipass
multipass find # find Ubuntu versions
multipass launch jammy --name hadoop-vm # Ubuntu 22.04 LTS

multipass list
multipass info hadoop-vm
multipass shell hadoop-vm

multipass start hadoop-vm
multipass stop hadoop-vm

multipass delete hadoop-vm
multipass purge # for deletion

# Another option is: Lima (Linux machine)
brew install lima
limactl start
```
Mount data storage
```bash
# in MacOS
multipass mount . hadoop-vm:/home/ubuntu/Hadoop-VM
multipass umount Hadoop-VM # [instance-name]
```
## Manipulate on `multipass shell hadoop-vm`
Install SSH
```bash
sudo apt-get install openssh-server
sudo service ssh start
sudo service ssh status
sudo -i # change to root
sudo nano /etc/ssh/sshd_config
```
Uncomment `Port 22`
```bash
su - ubuntu
```
Install Java-8
```bash
sudo apt update
sudo apt install openjdk-8-jdk-headless
java -version
```
```bash
sudo nano /etc/environment
```
Add at the end of line: `export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-arm64"` (or `java-8-openjdk-amd64`).
```bash
source /etc/environment
code ~/.bashrc # open vscode, and for editting
```
Add at the end of file:
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-arm64
export PATH=$PATH:/usr/lib/jvm/java-8-openjdk-arm64/bin
export HADOOP_HOME=~/hadoop-3.2.3/
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
export HADOOP_STREAMING=$HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.2.3.jar
export HADOOP_LOG_DIR=$HADOOP_HOME/logs
export PDSH_RCMD_TYPE=ssh
```
```bash
source ~/.bashrc
echo $JAVA_HOME # Verify: show "/usr/lib/jvm/java-8-openjdk-amd64"

sudo adduser hadoop # (Optional) Create Hadoop User, Example password: 1234
su - hadoop # Activate hadoop user, enter password "1234", sudo -i fo `root` user
exit # Back to user `ubuntu` to store file

cd ~/Hadoop-VM
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.2.3/hadoop-3.2.3.tar.gz
tar -xzvf hadoop-3.2.3.tar.gz
cd hadoop-3.2.3
```
- In file `etc/hadoop/hadoop-env.sh` (access by `code $HADOOP_HOME/etc/hadoop/hadoop-env.sh`). Edit at line 37:
```
JAVA_HOME=/usr/lib/jvm/java-8-openjdk-arm64
```
- In file `etc/hadoop/core-site.xml` (access by `nano $HADOOP_HOME/etc/hadoop/core-site.xml`):
```
<configuration>
  <property>
    <name>fs.defaultFS</name> 
    <value>hdfs://localhost:9000</value> 
  </property> 
  <property>
    <name>hadoop.proxyuser.dataflair.groups</name> 
    <value>*</value> 
  </property>
  <property>
    <name>hadoop.proxyuser.dataflair.hosts</name> 
    <value>*</value> 
  </property>
  <property>
    <name>hadoop.proxyuser.server.hosts</name> 
    <value>*</value> 
  </property>
  <property>
    <name>hadoop.proxyuser.server.groups</name> 
    <value>*</value> 
  </property>
</configuration>
```
- In file `etc/hadoop/hdfs-site.xml`:
```
<configuration>
  <property> 
    <name>dfs.replication</name> 
    <value>1</value>
  </property>
</configuration>
```
- In file `etc/hadoop/mapred-site.xml`:
```
<configuration>
    <property> 
        <name>mapreduce.framework.name</name>
        <value>yarn</value> 
    </property> 
    <property>
        <name>mapreduce.application.classpath</name> 
        <value>$HADOOP_MAPRED_HOME/share/hado op/mapreduce/*:$HADOOP_MAPRED_HOME/sha re/hadoop/mapreduce/lib/*</value>
    </property> 
</configuration>
```
- In file `etc/hadoop/yarn-site.xml`:
```
<configuration>
    <property> 
        <name>yarn.nodemanager.aux-services</name> 
        <value>mapreduce_shuffle</value>
    </property>
    <property> 
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOM E,HADOOP_HDFS_HOME,HADOOP_CONF_DIR ,CLASSPATH_PREP END_DISTCACHE,HADOOP_YARN_HOME,HAD OOP_MAPRED_HOME</value>
    </property> 
</configuration>
```
Setup SSH for Hadoop (Ubuntu) User
```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```
Copy the Hadoop installation to a location outside the **shared/mounted** folder
```bash
cp -r ~/Hadoop-VM/hadoop-3.2.3 ~/ # Or: `mv`
# export HADOOP_HOME=~/hadoop-3.2.3
# export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
# source ~/.bashrc # if changed in ~/.bashrc
```
# Process on HDFS
```bash
chmod +x ~/*    # (Optional)
hdfs namenode -format # or `hadoop-3.2.3/bin/hdfs namenode -format`
ls hadoop-3.2.3/sbin
start-all.sh    # combined: start-dfs.sh and start-yarn.sh
                # or: hadoop-3.2.3/sbin/start-all.sh
jps             # Output is similar this: 2417 NameNode, 2577 DataNode, 3298 NodeManager, 3730 Jps, 3111 ResourceManager, 2824 SecondaryNameNode
stop-all.sh

ip addr show    # Look for the IP address under the eth0 or ens3 interface. It will look something like 192.168.x.x or 10.0.x.x (e.g: 10.211.57.19)
```
- Go to
  - HDFS: http://10.211.57.19:9870 (http://localhost:9870)
  - YARN: http://10.211.57.19:8088 (http://localhost:8088)
```
NameNode                  : http://localhost:9870 (browse files)
NodeManager               : http://localhost:8042
Resource Manager (Yarn)   : http://localhost:8088/cluster
```

Access the HDFS Web UI (`http://<your_vm_ip>:9870`):
```bash
cd ~
hdfs dfs -mkdir -p /sales/data # hadoop fs -mkdir -p /sales/data
hdfs dfs -put Hadoop-VM/data/data.csv /sales/data
hdfs dfs -ls /sales/data

scp /path/to/your/file username@<vm_ip>:/path/in/vm/ # e.g: scp back-rlhf-chatgpt.jpg ubuntu@10.211.57.19:~/hadoop-vm/data

hdfs dfs -get /sales/data/data.csv ~/Hadoop-VM/data
hdfs dfs -rm -r /sales
hdfs dfs -mv oldname newname
```
Go to `http://<your_vm_ip>:9870/explorer.html` (the HDFS Web UI).

# (Optional) Open VScode for the Multipass instance
```bash
# On Ubuntu Multipass
sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh

# On macos
nano ~/.ssh/config
    ## Add this:
    Host hadoop-vm
    HostName 10.211.57.19 # can see in `multipass list`
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
# DO: Go to VScode -> Connect to Host -> Connect "hadoop-vm"
```
## Fix Error:
E.g: "user@instance-ip" is "ubuntu@10.211.57.19"
1. Error: Could not establish connection to "hadoop-vm": Permission denied (publickey).
```bash
ssh-copy-id ubuntu@10.211.57.19 # ubuntu@<instance-ip>, And: The bug "ubuntu@10.211.57.19: Permission denied (publickey)" bellow occurs
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/config
ssh ubuntu@10.211.57.19 # Check the ssh connection
```

1. Error: ubuntu@10.211.57.19: Permission denied (publickey).
```bash
cat ~/.ssh/id_ed25519.pub # E.g you public key: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKaTlX2MjUu0w962dPQMoq3Gq6Y3Ec5la2qh3SVu4R8k bdbao@bdbaos-MBP.local
(ssh-keygen -t ed25519) # if you don't have one

multipass shell hadoop-vm
    mkdir -p ~/.ssh
    echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKaTlX2MjUu0w962dPQMoq3Gq6Y3Ec5la2qh3SVu4R8k bdbao@bdbaos-MBP.local" >> ~/.ssh/authorized_keys # "your-public-key-here"
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys

    sudo systemctl start ssh # Ensure the SSH service is running
    sudo systemctl enable ssh

ssh ubuntu@10.211.57.19 # Check the ssh connection
```
