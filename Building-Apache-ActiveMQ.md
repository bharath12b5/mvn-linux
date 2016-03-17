<!---PACKAGE:Apache ActiveMQ--->
<!---DISTRO:SLES 12:5.13.1--->
<!---DISTRO:RHEL 7.1:5.13.1--->

# Building Apache ActiveMQ

[Apache ActiveMQ](http://activemq.apache.org/) is an open source message broker written in Java together with a full Java Message Service (JMS) client. The stable release of ActiveMQ 5.13.1 has been built and tested on Linux on z Systems.  The following instructions can be used for RHEL 7.1 and SLES 12.

### Prerequisites:
   * Maven v3.0.0 or above
   -- Instructions for building Maven can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven).

   _**Note**_: Maven location is required to build AcitveMQ (see Step 2 below).

### _**General Note:**_
i)  _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory /\<source_root\>/ will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

## Building and Installing Apache ActiveMQ

### Step 1: Install the Dependencies

For RHEL 7.1

    sudo yum install git java-1.7.0-openjdk-devel.s390x tar
  
For SLES 12

    sudo zypper install git java-1.7.0-openjdk tar



### Step 2: Set environment variables and working directory

    cd /<source_root>/
    export WORK_DIR=`pwd`
    export JAVA_HOME=/usr/lib64/jvm/java
    export M2_HOME=/<Your Maven location>/
    export MAVEN_OPTS="-Xms512m -Xmx1024m"
    export PATH=$JAVA_HOME/bin:$PATH:$M2_HOME/bin

### Step 3: Build and install Apache ActiveMQ 5.13.1

    cd /<source_root>/
    git clone -b activemq-5.13.1 https://github.com/apache/activemq.git
    cd activemq
    mvn clean install -Dmaven.test.skip=true

### Step 4: Retrieve Apache ActiveMQ from repository

   Maven install from Step 3 created a tar file in a local repository which is set to `˜/.m2` by default.

    cd /<source_root>/
    tar -xzf ˜/.m2/repository/org/apache/activemq/apache-activemq/5.13.1/apache-activemq-5.13.1-bin.tar.gz

   The binary activemq will be available at ` /<source_root>/apache-activemq-5.13.1/bin`

### Step 5: Smoke Test (Optional)
    cd /<source_root>/activemq
    mvn -Dactivemq.tests=smoke test
  
### Step 6: Verify Apache ActiveMQ is starting correctly

1.   Start ApacheMQ
    ```
    cd /<source_root>/apache-activemq-5.13.1/bin
    ./activemq start
    ```
  
1.   The log file at ` /<source_root>/data/activemq.log ` should have a line like below, confirming ApacheMQ has started successfully

    ```
    2016-03-09 14:54:14,859 | INFO  | Apache ActiveMQ 5.13.1 (localhost, ID:xxxxxxxx-46646-1457553254202-0:1) started | org.apache.activemq.broker.BrokerService | main
    ```
   
1.   Get more ApacheMQ status

    ```
    ./activemq bstat
    ```
  
1.   Stop ApacheMQ

    ```
    ./activemq stop
    ```
    
## Reference
http://activemq.apache.org