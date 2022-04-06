
<h3>1- Installing Java</h3>
First we need to install java 11  
We prefer to install with apt-get install  

```
sudo apt-get update
sudo apt-get install openjdk-11-jdk
```
find java directory where java installed  
```
update-alternatives --config java
```
/usr/lib/jvm/java-11-openjdk-amd64/bin/java


add java_home and path variables to the java.sh file 
```
sudo vi /etc/profile.d/java.sh
```
add these lines inside java.sh
```
#/bin/bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
```
apply changes  
```
source /etc/profile.d/java.sh
```
check java version
```
java --version
```
you have to get output like this
```
openjdk 11.0.13 2021-10-19
OpenJDK Runtime Environment (build 11.0.13+8-Ubuntu-0ubuntu1.20.04)
OpenJDK 64-Bit Server VM (build 11.0.13+8-Ubuntu-0ubuntu1.20.04, mixed mode, sharing)
```

<h3>2- Installing Nifi</h3>

download nifi and upload to the server  
I downloaded lastest version "nifi-1.15.3-bin.tar.gz"  
<url>https://nifi.apache.org/download.html  

move file to the secure area
```
mv /home/dtpamdatalake01/nifi-1.15.3-bin.tar.gz /data/downloads/
```
extract files in which path you want to install
```
tar xvf /data/downloads/nifi-1.15.3-bin.tar.gz -C /data/
```
go inside nifi directory
```
cd /data/nifi-1.15.3
```
Be careful, there are no zookeeper installed before and zookeper port is available.
You can check with this command
```
sudo lsof -i:2181 | grep LISTEN
```
define zookeeper properties.
```
vi conf/zookeeper.properties
```
```
initLimit=10
autopurge.purgeInterval=24
syncLimit=5
tickTime=2000
dataDir=./state/zookeeper
autopurge.snapRetainCount=30
server.1=192.168.000.000:2888:3888;2181
server.2=192.168.000.001:2888:3888;2181
server.3=192.168.000.002:2888:3888;2181
```
create directories
```
mkdir state
mkdir state/zookeeper
```
define myid file as below starting from 1 and increase with +1  
for machine 1: ``echo 1 > state/zookeeper/myid``  
for machine 2: ``echo 2 > state/zookeeper/myid``  
for machine 3: ``echo 3 > state/zookeeper/myid``  


Set state managment connection string.  
Change *property name="Connect String"* inside *cluster-provider*
```
vi conf/state-management.xml
```
```
<property name="Connect String">192.168.000.000:2181,192.168.000.001:2181,192.168.000.002:2181</property>
```

set nifi properties for each machine with its ip.
```
vi conf/nifi.properties
```

```
nifi.state.management.embedded.zookeeper.start=true
nifi.remote.input.host=192.168.000.000
nifi.remote.input.secure=false
nifi.remote.input.socket.port=10000
nifi.web.http.host=192.168.000.000
nifi.web.http.port=7070
nifi.web.https.host=
nifi.web.https.port=
nifi.sensitive.props.key=l5bGW7Miy5Vv5sGTr8tXqLyVfpOdTnY0
nifi.security.keystore=
nifi.security.keystoreType=
nifi.security.truststore=
nifi.security.truststoreType=
nifi.cluster.is.node=true
nifi.cluster.node.address=192.168.000.000
nifi.cluster.node.protocol.port=9991
nifi.zookeeper.connect.string=192.168.000.000:2181,192.168.000.001:2181,192.168.000.002:2181
```

start nifi with this command
```
/data/nifi-1.15.3/bin/nifi.sh start
```
check log for application status or errors
```
tail -100f logs/nifi-app.log
```
after 2-3 min you can open the link inside browser  
<url>http://192.168.000.000:7070/

apply to use nifi as a service
```
/data/nifi-1.15.3/bin/nifi.sh install
sudo systemctl daemon-reload
sudo service nifi start
```
check service status
```
sudo service nifi status
```
