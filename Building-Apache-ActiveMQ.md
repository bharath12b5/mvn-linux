<!---PACKAGE:Apache ActiveMQ--->
<!---DISTRO:SLES 12:5.14.0--->
<!---DISTRO:RHEL 7.1:5.14.0--->
<!---DISTRO:Ubuntu 16.x:Distro,5.14.0--->

# Building Apache ActiveMQ

The below versions are available in respective Distributions at the time of this recipe Creation. 

• Ubuntu 16.04 has `5.13.2`

# **Building Apache ActiveMQ**
[Apache ActiveMQ](http://activemq.apache.org/) is an open source message broker written in Java together with a full Java Message Service (JMS) client. The stable release of ActiveMQ 5.14.0 has been built and tested on Linux on z Systems. The following instructions can be used for RHEL 7.1, SLES 12 and Ubuntu 16.04.

###Prerequisites: ###
*    Maven v3.0.0 or above -- Instructions for building Maven can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven).

    _**Note:**_ Maven location is required to build AcitveMQ (see Step 2 below).

### **_General Notes:_**

_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory /<source_root>/ will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

## Building and Installing Apache ActiveMQ

### Step 1: Install the Dependencies

For RHEL 7.1

```
sudo yum install java-1.7.0-openjdk-devel tar wget git 
```

For SLES 12

```
sudo zypper install java-1_7_0-openjdk-devel tar wget git
```

For Ubuntu 16.04

```
sudo apt-get update
sudo apt-get install maven git openjdk-8-jdk
```

### Step 2: Set JAVA_HOME
For RHEL 7.1
```
export JAVA_HOME=/usr/lib/jvm/java
```
For SLES 12
```
export JAVA_HOME=/usr/lib64/jvm/java-openjdk
```

For Ubuntu 16.04
```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-s390x
```
### Step 3: Set environment variables and working directory
```
cd /<source_root>/
export WORK_DIR=`pwd`
export MAVEN_OPTS="-Xms512m -Xmx1024m"
export PATH=$JAVA_HOME/bin:$PATH
```
### Step 4: Build and install Apache ActiveMQ 5.14.0

```
cd /<source_root>/
git clone -b activemq-5.14.0 https://github.com/apache/activemq.git
cd activemq
mvn clean install -Dmaven.test.skip=true
```

### Step 5: Retrieve Apache ActiveMQ from repository
Maven install from Step 3 created a tar file in a local repository which is set to ~/.m2 by default.
```
cd /<source_root>/
tar -xzf ~/.m2/repository/org/apache/activemq/apache-activemq/5.14.0/apache-activemq-5.14.0-bin.tar.gz
```
The binary activemq will be available at `/<source_root>/apache-activemq-5.14.0/bin`

### Step 6: Smoke Test (Optional)
```
cd /<source_root>/activemq
mvn -Dactivemq.tests=smoke test
```

### Step 7: Verify Apache ActiveMQ is starting correctly
1.   Start ApacheMQ
    ```
    cd /<source_root>/apache-activemq-5.14.0/bin
    ./activemq start
    ```
  
1.   The log file at ` /<source_root>/apache-activemq-5.14.0/data/activemq.log ` should have a line like below, confirming ApacheMQ has started successfully

    ```
    2016-09-20 08:21:01,329 | INFO  | Apache ActiveMQ 5.14.0 (localhost, ID:1cf5a6e8c230-46325-1474359660575-0:1) started | org.apache.activemq.broker.BrokerService | main
    ```
   
1.   Get more ApacheMQ status

    ```
    ./activemq bstat
    ```
  
1.   Stop ApacheMQ

    ```
    ./activemq stop
    ```