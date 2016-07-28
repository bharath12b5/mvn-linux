<!---PACKAGE:Marathon--->
<!---DISTRO:SLES 12:1.1--->
<!---DISTRO:RHEL 7.1:1.1--->
<!---DISTRO:Ubuntu 16.x:1.1--->

# Building mesosphere/marathon
Mesosphere/marathon 1.1 has been successfully built on Linux on z Systems. The following instructions can be used on RHEL 7, SLES 12(SP1) and Ubuntu 16.04

##### General Notes:
i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory  ```/<source_root>/```  will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it.

### 1. Install the following dependencies
SLES12(SP1):
```
sudo zypper install -y git wget tar java-1_8_0-openjdk-devel patch which
```
RHEL7:
```
sudo yum install -y git tar wget java-1.8.0-openjdk-devel patch which
```
Ubuntu 16.04:
```
sudo apt-get install -y git tar wget openjdk-8-jdk patch
```
##### Set environment variables:

SLES12(SP1):
```
export JAVA_HOME=/usr/lib64/jvm/java-1.8.0
export PATH=/usr/lib64/jvm/java-1.8.0/bin:$PATH
```
RHEL7:
```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0
export PATH=/usr/lib/jvm/java-1.8.0/bin:$PATH
```
##### Install Sbt using the following steps

```
cd /<source_root>/
wget https://dl.bintray.com/sbt/native-packages/sbt/0.13.11/sbt-0.13.11.tgz
tar -zxvf sbt-0.13.11.tgz
export PATH=/<source_root>/sbt/bin:$PATH
```
Note: [Reference for sbt installation](http://www.scala-sbt.org/0.13/tutorial/Setup.html)

##### Install Apache Mesos
Apache Mesos installation instructions can be found from [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-Mesos).

### 2. Build Marathon

Check out the repo:
```
cd /<source_root>/
git clone https://github.com/mesosphere/marathon.git
cd marathon
git checkout releases/1.1
sbt assembly
```

Run ```./bin/build-distribution``` to package Marathon as an executable JAR (optional).


### 3. Start Marathon
Start Zookeeper service using following commands
```
cd /<source_root>/mesos/build/3rdparty/zookeeper-3.4.5
cp conf/zoo_sample.cfg conf/zoo.cfg
sudo ./bin/zkServer.sh start
```

Start Marathon master
```
cd /<source_root>/marathon
./bin/start --master local --zk zk://<ip_address>:2181/marathon
```

Open ```http://<ip_address>:8080``` in your browser to access Web UI.
### References
For more information on how to run Marathon in production and configuration options, see the [Marathon docs](https://mesosphere.github.io/marathon/docs/).