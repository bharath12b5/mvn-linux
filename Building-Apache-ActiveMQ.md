<!---PACKAGE:Apache ActiveMQ--->
<!---DISTRO:SLES 12:5.14.3--->
<!---DISTRO:RHEL 7.1:5.14.3--->
<!---DISTRO:Ubuntu 16.x:Distro,5.14.3--->

# Building Apache ActiveMQ

The below versions are available in respective Distributions at the time of this recipe Creation. 

* Ubuntu 16.04 has `5.13.2`
* Ubuntu 16.10 has `5.14.0`

[Apache ActiveMQ](http://activemq.apache.org/) is an open source message broker written in Java together with a full Java Message Service (JMS) client. The stable release of ActiveMQ 5.14.3 has been built and tested on Linux on z Systems. The following instructions can be used for RHEL 7.1/7.2/7.3, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

###Prerequisites: ###
*    Maven v3.0.0 or above -- Instructions for building Maven can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven).

    _**Note:**_ Maven location is required to build AcitveMQ, please make sure `mvn` is in PATH.

### **_General Notes:_**

* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory /<source_root>/ will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

##Step 1: Building and Installing Apache ActiveMQ

####1.1) Install the Dependencies

* RHEL 7.1/7.2/7.3

  ```
  sudo yum install java-1.7.0-openjdk-devel tar wget git 
  ```

* SLES 12/12-SP1/12-SP2

  ```
  sudo zypper install java-1_7_0-openjdk-devel tar wget git
  ```

* Ubuntu 16.04/16.10

  ```
  sudo apt-get update
  sudo apt-get install maven git openjdk-8-jdk
  ```

####1.2) Set JAVA_HOME

* RHEL 7.1/7.2/7.3

  ```
  export JAVA_HOME=/usr/lib/jvm/java
  ```
* SLES 12/12-SP1/12-SP2
  ```
  export JAVA_HOME=/usr/lib64/jvm/java-openjdk
  ```

* Ubuntu 16.04/16.10
  ```
  export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-s390x
  ```
  
####1.3) Set environment variables and working directory
  ```
  cd /<source_root>/
  export WORK_DIR=`pwd`
  export MAVEN_OPTS="-Xms512m -Xmx1024m"
  export PATH=$JAVA_HOME/bin:$PATH
  ```
  
####1.4) Build and install Apache ActiveMQ 5.14.3

  ```
  cd /<source_root>/
  git clone -b activemq-5.14.3 https://github.com/apache/activemq.git
  cd activemq
  mvn clean install -Dmaven.test.skip=true
  ```

####1.5) Retrieve Apache ActiveMQ from repository
  Maven install from Step 3 created a tar file in a local repository which is set to ~/.m2 by default.
  ```
  cd /<source_root>/
  tar -xzf ~/.m2/repository/org/apache/activemq/apache-activemq/5.14.3/apache-activemq-5.14.3-bin.tar.gz
  ```
  The binary activemq will be available at `/<source_root>/apache-activemq-5.14.3/bin`
## Step 2: Testing

####2.1) Smoke Test (Optional)
```
cd /<source_root>/activemq
mvn -Dactivemq.tests=smoke test
```

####2.2) Verify Apache ActiveMQ is starting correctly
* Start ApacheMQ
    ```
    cd /<source_root>/apache-activemq-5.14.3/bin
    ./activemq start
    ```
  
* The log file at ` /<source_root>/apache-activemq-5.14.3/data/activemq.log ` should have a line like below, confirming ApacheMQ has started successfully

    ```
    2016-12-20 08:21:01,329 | INFO  | Apache ActiveMQ 5.14.3 (localhost, ID:1cf5a6e8c230-46325-1474359660575-0:1) started | org.apache.activemq.broker.BrokerService | main
    ```
   
* Get more ApacheMQ status

    ```
    ./activemq bstat
    ```
  
* Stop ApacheMQ

    ```
    ./activemq stop
    ```

##References:
http://activemq.apache.org/
