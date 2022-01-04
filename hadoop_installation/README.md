# Hadoop 安裝流程

## 登入主機


帳號密碼跟連接埠請參考Google Sheets

連線到master主機:
```console
ssh your_master_username@ip -p master_ssh_port
```

連線到slave主機:
```console
ssh your_slave_username@ip -p master_ssh_port
```


## Hadoop 安裝

### Step 1 修改hosts file

修改每台主機(包含master和slave)的hosts文件，讓主機可以直接認得該host的IP。

查看該主機IP
```console
ifconfig
```

結果如下，可以知道master主機的IP為`172.17.0.45`，然後也在slave用同樣的指令取得IP
```console
5724_udic_hadoop_master_0@5724_udic_hadoop_master_0:~$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.45  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:2d  txqueuelen 0  (Ethernet)
        RX packets 110  bytes 21844 (21.8 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 98  bytes 17489 (17.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

安裝`nano`編輯器 (會使用`vim`的同學可以不用安裝，之後需要使用編輯器的步驟直接改用`vim`)
```console
sudo apt update
sudo apt install nano
```
執行`sudo`指令時需要再輸入一次密碼:
```console
5724_udic_hadoop_master_0@5724_udic_hadoop_master_0:~$ sudo vim /etc/hosts
[sudo] password for 5724_udic_hadoop_master_0:
```

新增slave和master的hostname讓他們認得彼此的IP。(master和slave都要進行同樣操作)
```console
sudo nano /etc/hosts
```



在master的檔案中加入slave的IP，在slave的檔案中加入master的IP，並加入master、slave，如下

master修改後的hosts file:
```
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.45     master  5724_udic_hadoop_master_0
172.17.0.46     slave
```

slave修改後的hosts file:
```
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.46     slave   5724_udic_hadoop_slave_0-0
172.17.0.45     master
```
修改完成後存檔離開文件

### Step 2 ping測試

修改完hosts文件後可以互`ping`看看是否正常。(Ctrl+C停止)

```console
5724_udic_hadoop_master_0@5724_udic_hadoop_master_0:~$ ping slave
PING slave (172.17.0.46) 56(84) bytes of data.
64 bytes from slave (172.17.0.46): icmp_seq=1 ttl=64 time=0.423 ms
64 bytes from slave (172.17.0.46): icmp_seq=2 ttl=64 time=0.076 ms
64 bytes from slave (172.17.0.46): icmp_seq=3 ttl=64 time=0.070 ms
64 bytes from slave (172.17.0.46): icmp_seq=4 ttl=64 time=0.067 ms

--- slave ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3064ms
rtt min/avg/max/mdev = 0.067/0.159/0.423/0.152 ms
```

```console
5724_udic_hadoop_slave_0-0@5724_udic_hadoop_slave_0-0:~$ ping master
PING master (172.17.0.45) 56(84) bytes of data.
64 bytes from master (172.17.0.45): icmp_seq=1 ttl=64 time=0.097 ms
64 bytes from master (172.17.0.45): icmp_seq=2 ttl=64 time=0.071 ms
64 bytes from master (172.17.0.45): icmp_seq=3 ttl=64 time=0.070 ms
64 bytes from master (172.17.0.45): icmp_seq=4 ttl=64 time=0.063 ms

--- master ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3054ms
rtt min/avg/max/mdev = 0.063/0.075/0.097/0.014 ms
```

### Step 3 新增hduser

**master和slave都要做**

新增hadoop使用者(這邊都用hduser方便後續進行)
```console
sudo adduser hduser
```
過程如下，除了密碼要設定之外其他可以都直接按enter:
```console
5724_udic_hadoop_master_0@5724_udic_hadoop_master_0:~$ sudo adduser hduser
[sudo] password for 5724_udic_hadoop_master_0:
Adding user `hduser' ...
Adding new group `hduser' (1001) ...
Adding new user `hduser' (1001) with group `hduser' ...
Creating home directory `/home/hduser' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for hduser
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]
```

給予 sudo 權限
```console
sudo usermod -a -G sudo hduser
```

新增並加入hadoop群組
```console
sudo addgroup hadoop
sudo usermod -a -G hadoop hduser
```

將使用者切換到hduser
```console
su hduser
```

`groups`: 確認是否有sudo權限及所屬群組
```console
hduser@5724_udic_hadoop_master_0:/home/5724_udic_hadoop_master_0$ groups
hduser sudo hadoop
```
之後都會以`hduser`進行操作

### Step 4 產生公私鑰

在master產生公鑰與私鑰。
```console
ssh-keygen -t rsa
```
過程一直按Enter直到結束，如下圖。

```console
hduser@5724_udic_hadoop_master_0:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hduser/.ssh/id_rsa):
Created directory '/home/hduser/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/hduser/.ssh/id_rsa.
Your public key has been saved in /home/hduser/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:TEZxU2sOTUvqG83bLCqamg8v5VWV6pU13rkdmCWtN+A hduser@5724_udic_hadoop_master_0
The key's randomart image is:
+---[RSA 2048]----+
|        o.o.+..  |
|       . . *o=oo |
|        o oo*+Bo.|
|       + .o*oE.=.|
|        Soo.+ . =|
|      . . .o + ..|
|    .o .  . o o  |
|    .+...  . .   |
|    o+=. ..      |
+----[SHA256]-----+
```

### Step 5 傳送公私鑰

產生後在`.ssh`資料夾內會新增公鑰跟私鑰，如下，`id_rsa`為私鑰，`id_rsa.pub`為公鑰:
```console
hduser@5724_udic_hadoop_master_0:~$ cd ~/.ssh
hduser@5724_udic_hadoop_master_0:~/.ssh$ ls
id_rsa  id_rsa.pub
```

將公鑰加入`authorized_keys`檔案中
```console
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

檢查公鑰是否有加入`authorized_keys`
```console
cat ~/.ssh/authorized_keys
```

將新增的公鑰與私鑰傳給slave
```console
scp -r ~/.ssh hduser@slave:~/.ssh
```
傳送過程如下:
```console
hduser@5724_udic_hadoop_master_0:~/.ssh$ scp -r ~/.ssh hduser@slave:~/.ssh
The authenticity of host 'slave (172.17.0.46)' can't be established.
ECDSA key fingerprint is SHA256:6sXUeozIiyuXZCSm639jvRc1rD7fDCTK2pjaGKVxFUY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'slave,172.17.0.46' (ECDSA) to the list of known hosts.
hduser@slave's password:
id_rsa.pub                                             100%  414     1.2MB/s   00:00
authorized_keys                                        100%  414     1.2MB/s   00:00
known_hosts                                            100%  444     2.6MB/s   00:00
id_rsa                                                 100% 1679    14.9MB/s   00:00
```


### Step 6 測試無密碼登入

若成功則可以測試是否能夠無密碼切換不同主機(包含自己ssh自己，master和slave端都要測)，指令如下。

登入master
```console
ssh master
```
登入slave
```console
ssh slave
```
登出
```console
exit
```

例1: 在master無密碼登入slave
```console
hduser@5724_udic_hadoop_master_0:~$ ssh slave
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 5.4.0-74-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
hduser@5724_udic_hadoop_slave_0-0:~$ exit
logout
Connection to slave closed.
hduser@5724_udic_hadoop_master_0:~$
```
例2: 在slave無密碼登入master
```console
hduser@5724_udic_hadoop_slave_0-0:~$ ssh master
The authenticity of host 'master (172.17.0.45)' can't be established.
ECDSA key fingerprint is SHA256:6sXUeozIiyuXZCSm639jvRc1rD7fDCTK2pjaGKVxFUY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'master,172.17.0.45' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 5.4.0-74-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

hduser@5724_udic_hadoop_master_0:~$ exit
logout
Connection to master closed.
hduser@5724_udic_hadoop_slave_0-0:~$
```
例3: 在master無密碼登入master
```console
hduser@5724_udic_hadoop_master_0:~$ ssh master
The authenticity of host 'master (172.17.0.45)' can't be established.
ECDSA key fingerprint is SHA256:6sXUeozIiyuXZCSm639jvRc1rD7fDCTK2pjaGKVxFUY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'master,172.17.0.45' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 5.4.0-74-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Tue Jan  4 09:08:19 2022 from 172.17.0.46
hduser@5724_udic_hadoop_master_0:~$ exit
logout
Connection to master closed.
hduser@5724_udic_hadoop_master_0:~$
```

### Step 7

以下步驟(Step 8~Step 11)在`master`端完成後，在`slave`端完成相同的修改後再繼續。

### Step 8 安裝java、下載hadoop

安裝`java`
```console
sudo apt install openjdk-8-jdk
```

下載hadoop2.7.7版本
```console
cd ~
wget https://www.dropbox.com/s/qbvuoevst1mmdnf/hadoop-2.7.7.tar.gz
```

### Step 9 移動hadoop目錄

下載完後解壓縮檔案
```console
tar zxvf hadoop-2.7.7.tar.gz
```

將`hadoop-2.7.7`移動到`/usr/local/`底下，並重新命名為`hadoop`。
```console
sudo mv hadoop-2.7.7 /usr/local/hadoop
```

在`/usr/local`底下查看hadoop資料夾是否正確移動
```console
ls /usr/local/
```
結果如下，看到`hadoop`已經正確移動到`/usr/local`底下:
```console
hduser@5724_udic_hadoop_master_0:~$ ls /usr/local/
bin  cuda  cuda-10.0  etc  games  hadoop  include  lib  man  sbin  share  src
```

### Step 10 設定環境變數

編輯`.bashrc`file，方便我們打指令
```console
nano ~/.bashrc
```
在檔案中任何一處增加以下環境變數設定，貼上後儲存離開
```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin 
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native:$JAVA_LIBRARY_PATH
export CLASSPATH=$(hadoop classpath)
```

### Step 11 測試環境變數

使設定檔生效
```console
source ~/.bashrc
```

確認環境變數是否成功更改
```console
hadoop
```
若輸出結果如下代表環境變數已成功更改:
```console
hduser@5724_udic_hadoop_master_0:~$ hadoop
Usage: hadoop [--config confdir] [COMMAND | CLASSNAME]
  CLASSNAME            run the class named CLASSNAME
 or
  where COMMAND is one of:
  fs                   run a generic filesystem user client
  version              print the version
  jar <jar>            run a jar file
                       note: please use "yarn jar" to launch
                             YARN applications, not this command.
  checknative [-a|-h]  check native hadoop and compression libraries availability
  distcp <srcurl> <desturl> copy file or directories recursively
  archive -archiveName NAME -p <parent path> <src>* <dest> create a hadoop archive
  classpath            prints the class path needed to get the
  credential           interact with credential providers
                       Hadoop jar and the required libraries
  daemonlog            get/set the log level for each daemon
  trace                view and modify Hadoop tracing settings

Most commands print help when invoked w/o parameters.
```
若出現`hadoop: command not found`，則檢查hadoop資料夾是否在/usr/local底下以及是否記得運行Step9之步驟
```console
5724_udic_hadoop_master_1@5724_udic_hadoop_master_1:~$ hadoop
-bash: hadoop: command not found
```

接下來在slave上也操作Step 8~Step 11再繼續

### Step 12 列出hadoop文件

接下來的步驟都在master完成，以下將要修改hadoop的文件，進入hadoop資料夾後需要設置的有七個檔案：`hadoop-env.sh`、`yarn-env.sh`、`slaves`、`core-site.xml`、`hdfs-site.xml`、`maprd-site.xml`、`yarn-site.xml`。
```console
cd /usr/local/hadoop/etc/hadoop/
ls
```
結果:
```console
hduser@5724_udic_hadoop_master_0:~$ cd /usr/local/hadoop/etc/hadoop/
hduser@5724_udic_hadoop_master_0:/usr/local/hadoop/etc/hadoop$ ls
capacity-scheduler.xml      hadoop-policy.xml        kms-log4j.properties        ssl-client.xml.example
configuration.xsl           hdfs-site.xml            kms-site.xml                ssl-server.xml.example
container-executor.cfg      httpfs-env.sh            log4j.properties            yarn-env.cmd
core-site.xml               httpfs-log4j.properties  mapred-env.cmd              yarn-env.sh
hadoop-env.cmd              httpfs-signature.secret  mapred-env.sh               yarn-site.xml
hadoop-env.sh               httpfs-site.xml          mapred-queues.xml.template
hadoop-metrics2.properties  kms-acls.xml             mapred-site.xml.template
hadoop-metrics.properties   kms-env.sh               slaves
```

### Step 13 修改`hadoop-env.sh`

`hadoop-env.sh`的修改
```console
sudo nano hadoop-env.sh
```

將以下檔案片段中第3行的`export JAVA_HOME=${JAVA_HOME}`取代成`export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64`
```bash
...
# The java implementation to use.
export JAVA_HOME=${JAVA_HOME}      <------這一行

# The jsvc implementation to use. Jsvc is required to run secure datanodes
# that bind to privileged ports to provide authentication of data transfer
# protocol.  Jsvc is not required if SASL is configured for authentication of
# data transfer protocol using non-privileged ports.
#export JSVC_HOME=${JSVC_HOME}

export HADOOP_CONF_DIR=${HADOOP_CONF_DIR:-"/etc/hadoop"}

# Extra Java CLASSPATH elements.  Automatically insert capacity-scheduler.
for f in $HADOOP_HOME/contrib/capacity-scheduler/*.jar; do
  if [ "$HADOOP_CLASSPATH" ]; then
    export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$f
  else
    export HADOOP_CLASSPATH=$f
  fi
done
...
```
修改後儲存離開

### Step 14 修改`yarn-env.sh`

`yarn-env.sh`的修改
```console
sudo nano yarn-env.sh
```

將以下檔案片段中第9行的`# export JAVA_HOME=/home/y/libexec/jdk1.6.0/`取代成`export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64` (取消註解)
```bash
...
# User for YARN daemons
export HADOOP_YARN_USER=${HADOOP_YARN_USER:-yarn}

# resolve links - $0 may be a softlink
export YARN_CONF_DIR="${YARN_CONF_DIR:-$HADOOP_YARN_HOME/conf}"

# some Java parameters
# export JAVA_HOME=/home/y/libexec/jdk1.6.0/      <------這一行
if [ "$JAVA_HOME" != "" ]; then
  #echo "run java in $JAVA_HOME"
  JAVA_HOME=$JAVA_HOME
fi

if [ "$JAVA_HOME" = "" ]; then
  echo "Error: JAVA_HOME is not set."
  exit 1
fi

JAVA=$JAVA_HOME/bin/java
JAVA_HEAP_MAX=-Xmx1000m
...
```
修改後儲存離開

### Step 15 修改`slaves`

`slaves`的修改
這邊我們改成master和slave因為本次流程只有2台機器，我們讓他們都成為一個工作節點。
```console
sudo nano slaves
```

修改後的檔案:
```bash
# localhost
master
slave
```
修改後儲存離開

### Step 16 修改`core-site.xml`

`core-site.xml`的修改
```console
sudo nano core-site.xml
```

在configuration中加入property。
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000/</value>
    </property>
    <property>
         <name>hadoop.tmp.dir</name>
         <value>file:/usr/local/hadoop/tmp</value>
    </property>
</configuration>
```
修改後整個檔案長這樣:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000/</value>
    </property>
    <property>
         <name>hadoop.tmp.dir</name>
         <value>file:/usr/local/hadoop/tmp</value>
    </property>
</configuration>
```
修改後儲存離開

### Step 17 修改`hdfs-site.xml`

`hdfs-site.xml`的修改
```console
sudo nano hdfs-site.xml
```

同樣在configuration中加入property，參數dfs.replication為資料備份的數量，因我們只有2個node，若設大於2，結果會出錯。
```xml
<configuration>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>master:9001</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/hadoop_data/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/hadoop_data/hdfs/datanode</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>mapreduce.job.ubertask.enable</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>true</value>
    </property>
</configuration>
```

修改後整個檔案長這樣:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>master:9001</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/hadoop_data/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/hadoop_data/hdfs/datanode</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>mapreduce.job.ubertask.enable</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>true</value>
    </property>
</configuration>
```
修改後儲存離開

### Step 18 修改`mapred-site.xml`

`mapred-site.xml`的修改

若資料夾中沒有`mapred-site.xml`，請在資料夾中找到`mapred-site.xml.template`，並複製一份命名為`mapred-site.xml`來作修改。
```console
sudo cp /usr/local/hadoop/etc/hadoop/mapred-site.xml.template /usr/local/hadoop/etc/hadoop/mapred-site.xml
```

```console
sudo nano mapred-site.xml
```
在configuration中加入property
```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
修改後整個檔案長這樣:
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
修改後儲存離開

### Step 19 修改`yarn-site.xml`

`yarn-site.xml`的修改
```console
sudo nano yarn-site.xml
```
在configuration中加入property
```xml
<configuration> 
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>2048</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>2</value>
    </property>
	<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>master:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>master:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>master:8035</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>master:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>master:8088</value>
    </property>
</configuration>
```
修改後整個檔案長這樣:
```xml
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>2048</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>2</value>
    </property>
        <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>master:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>master:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>master:8035</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>master:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>master:8088</value>
    </property>
</configuration>
```
修改後儲存離開

### Step 20 設定slave

把所有設定檔傳到slave，這樣設定slave時就不用再修改一次
```console
scp -r /usr/local/hadoop/etc/hadoop hduser@slave:~/hadoop-config
```

接下來在slave上操作，將master傳送過來的設定檔複製到hadoop目錄底下
```console
cp ~/hadoop-config/* /usr/local/hadoop/etc/hadoop/
```

### Step 21 格式化namenode

都配置完成後建立`namenode`和`datanode`資料夾，並且將hadoop資料夾的擁有者設為hduser。**(master和slave都要)**
```console
sudo mkdir -p /usr/local/hadoop/hadoop_data/hdfs/namenode
sudo mkdir -p /usr/local/hadoop/hadoop_data/hdfs/datanode
sudo chown hduser:sudo -R /usr/local/hadoop
```

做完以上的步驟，接來下在master格式化namenode
```console
hadoop namenode -format
```
執行結果節錄如下:
```console
...
22/01/04 11:09:48 INFO namenode.FSNamesystem: dfs.namenode.safemode.threshold-pct = 0.9990000128746033
22/01/04 11:09:48 INFO namenode.FSNamesystem: dfs.namenode.safemode.min.datanodes = 0
22/01/04 11:09:48 INFO namenode.FSNamesystem: dfs.namenode.safemode.extension     = 30000
22/01/04 11:09:48 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.window.num.buckets = 10
22/01/04 11:09:48 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.num.users = 10
22/01/04 11:09:48 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.windows.minutes = 1,5,25
22/01/04 11:09:48 INFO namenode.FSNamesystem: Retry cache on namenode is enabled
22/01/04 11:09:48 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis
22/01/04 11:09:48 INFO util.GSet: Computing capacity for map NameNodeRetryCache
22/01/04 11:09:48 INFO util.GSet: VM type       = 64-bit
22/01/04 11:09:48 INFO util.GSet: 0.029999999329447746% max memory 889 MB = 273.1 KB
22/01/04 11:09:48 INFO util.GSet: capacity      = 2^15 = 32768 entries
22/01/04 11:09:48 INFO namenode.FSImage: Allocated new BlockPoolId: BP-138878272-172.17.0.45-1641294588480
22/01/04 11:09:48 INFO common.Storage: Storage directory /usr/local/hadoop/hadoop_data/hdfs/namenode has been successfully formatted.
22/01/04 11:09:48 INFO namenode.FSImageFormatProtobuf: Saving image file /usr/local/hadoop/hadoop_data/hdfs/namenode/current/fsimage.ckpt_0000000000000000000 using no compression
22/01/04 11:09:48 INFO namenode.FSImageFormatProtobuf: Image file /usr/local/hadoop/hadoop_data/hdfs/namenode/current/fsimage.ckpt_0000000000000000000 of size 323 bytes saved in 0 seconds.
22/01/04 11:09:48 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
22/01/04 11:09:48 INFO util.ExitUtil: Exiting with status 0
22/01/04 11:09:48 INFO namenode.NameNode: SHUTDOWN_MSG:
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at master/172.17.0.45
************************************************************/
```

### Step 22 啟動hadoop

master 格式化後繼續下列指令啟動dfs和yarn。
```console
start-all.sh
```
執行結果如下，觀察輸出如果有某部分啟動失敗，再檢查先前是否有步驟沒做到:
```console
hduser@5724_udic_hadoop_master_0:/usr/local/hadoop/etc/hadoop$ start-all.sh
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
Starting namenodes on [master]
master: starting namenode, logging to /usr/local/hadoop/logs/hadoop-hduser-namenode-5724_udic_hadoop_master_0.out
slave: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-5724_udic_hadoop_slave_0-0.out
master: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-5724_udic_hadoop_master_0.out
Starting secondary namenodes [master]
master: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-hduser-secondarynamenode-5724_udic_hadoop_master_0.out
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop/logs/yarn-hduser-resourcemanager-5724_udic_hadoop_master_0.out
slave: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hduser-nodemanager-5724_udic_hadoop_slave_0-0.out
master: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hduser-nodemanager-5724_udic_hadoop_master_0.out
```
查看jps行程發現slave有的master都有，而master自己又額外多了一些進程，這是因為我們上面設定master除了負責管理以外自己也是一個slave。
```console
jps
```
master上的輸出:
```console
hduser@5724_udic_hadoop_master_0:/usr/local/hadoop/etc/hadoop$ jps
3618 DataNode
4466 Jps
3811 SecondaryNameNode
4298 NodeManager
3997 ResourceManager
3470 NameNode
```
slave上的輸出:
```console
hduser@5724_udic_hadoop_slave_0-0:/usr/local/hadoop/etc/hadoop$ jps
1793 NodeManager
1667 DataNode
1927 Jps
```

在hadoop上建立user目錄(在master端)
```console
hadoop fs -mkdir -p /user/hduser
```
修改hadoop tmp資料夾權限
```console
hadoop fs -mkdir -p /tmp
hadoop fs -chmod -R 777 /tmp
```

### Step 23 檢查WebUI

可以到瀏覽器上輸入 http://IP:WebUI Port 瀏覽Hadoop Resource Manager WebUI介面。

可以看到2個node
![](https://i.imgur.com/qkRgRki.png)


### Step 24 疑難排解

若jps行程跟Step 22的範例一模一樣的話，那大致上Hadoop已成功架設，若有不一樣的地方，你可以嘗試以下的方法。(master和slave都要)

在master上關閉dfs和yarn
```console
stop-all.sh
```
刪除hadoop_data資料夾
```console
sudo rm -r /usr/local/hadoop/hadoop_data
```
刪除tmp資料夾
```console
sudo rm -r /usr/local/hadoop/tmp
```
刪除logs資料夾
```console
sudo rm -r /usr/local/hadoop/logs
```

並重新運行Step 21~Step 23之步驟



## Ant 安裝

安裝ant讓主機可以編譯`src`目錄底下的java
```console
sudo apt install ant
```


讓JAVA 編譯中文時不會出錯
```console
sudo nano /etc/profile
```

把下面的參數貼上去，儲存後離開
```bash
export JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF8
```


使設定檔生效
```console
source /etc/profile
```
