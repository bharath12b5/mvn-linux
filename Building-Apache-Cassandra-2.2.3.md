### Building Apache Cassandra 2.2.3

Apache Cassandra is a scalable and fault-tolerant distributed NoSQL database with support for column indexes and denormalized collections. The stable release of Cassandra 2.2.3 has been built and tested on Linux on z Systems.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Building Apache Cassandra 2.2.3 with OpenJDK

The following build instructions have been tested with Apache Cassandra 2.2.3 on Linux on Z Systems with Java OpenJDK.

### Step 1: Install the dependencies:

* RHEL 7:
```
sudo yum install -y git which java-1.8.0-openjdk-devel.s390x gcc-c++ make automake autoconf libtool libstdc++-static tar wget patch words libXt-devel libX11-devel

```
* SLES12:
```
sudo zypper install -y java-1_7_0-openjdk-devel git which gcc-c++ make automake autoconf libtool libstdc++-devel tar wget patch words libXt-devel libX11-devel unzip xorg-x11-proto-devel xorg-x11-devel alsa-devel cups-devel
 
```
### Step 2: Set environment variables:
```    
unset JAVA_TOOL_OPTIONS
export LANG="en_US.UTF-8"
export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
```        
### Step 3: Install Ant:
```
cd /<source_root>/
wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.2-bin.tar.gz
tar -xvf apache-ant-1.9.2-bin.tar.gz
cd apache-ant-1.9.2
export ANT_HOME=`pwd`
cd bin
export PATH=$PATH:`pwd`
```
### Step 4: Install the latest version of Snappy-Java:
 ```
cd /<source_root>/
git clone https://github.com/xerial/snappy-java.git
cd snappy-java
git checkout develop
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk (Only for RHEL7)
export JAVA_HOME=/usr/lib64/jvm/java-1.7.0 (Only for SLES12)
make IBM_JDK_7=1 USE_GIT=1 GIT_SNAPPY_BRANCH=master GIT_REPO_URL=https://github.com/google/snappy.git
```        
### Step 5: Build and install Apache Cassandra 2.2.3:
```
cd /<source_root>/
wget https://archive.apache.org/dist/cassandra/2.2.3/apache-cassandra-2.2.3-src.tar.gz
tar -xvf apache-cassandra-2.2.3-src.tar.gz
cd apache-cassandra-2.2.3-src
```        
* Replace the original Snappy-Java jar file in the lib folder with installed Snappy-Java:
```
rm /<source_root>/apache-cassandra-2.2.3-src/lib/snappy-java-1.1.1.7.jar
cp /<source_root>/snappy-java/target/snappy-java-1.1.3-SNAPSHOT.jar /<source_root>/apache-cassandra-2.2.3-src/lib/snappy-java-1.1.3.jar
```    
* Build Apache Cassandra:
```
cd /<source_root>/apache-cassandra-2.2.3-src
ant
```  

_*Note:* The Apache Cassandra jar file is available under the bin folder._
    
### Step 6: (Optional) Run the unit tests:

* Build jna:
```
cd /<source_root>/
git clone https://github.com/java-native-access/jna.git
cd jna
ant
cp /<source_root>/jna/build/jna.jar  /<source_root>/apache-cassandra-2.2.3-src/lib/jna.jar
rm /<source_root>/apache-cassandra-2.2.3-src/lib/jna-4.0.0.jar

```
_*Note:*_ 
Use below commands to install IBM Java 1.7 and set it to Java Runtime Environment (Optional).

* RHEL 7:
```
sudo yum install java-1.7.1-ibm-devel.s390x
export JAVA_HOME=/usr/lib/jvm/java-1.7.1-ibm-1.7.1.3.10-1jpp.1.el7_1.s390x
sudo update-alternatives --config java    (Enter number to select IBM Java 1.7) 
```
* SLES 12:
```
sudo zypper install java-1_7_1-ibm-devel java-1_7_1-ibm-jdbc java-1_7_1-ibm
export JAVA_HOME=/usr/lib64/jvm/java-1.7.1-ibm
sudo update-alternatives --config java   (Enter number to select IBM Java 1.7)
```

Run the Cassandra test suite         
```
cd /<source_root>/apache-cassandra-2.2.3-src/
ant test
```        
_**Note:**_ 
_Run below steps to avoid test case failure in similar situation described below. Ignore `LegacySSTableTest` failure as it is not related to system Z_.

*  Replace time out value in build.xml file as shown below to avoid timeout error for multiple test cases.
```
        Older Value:
        <property name="test.timeout" value="60000" />
        <property name="test.long.timeout" value="600000" />
        <property name="test.burn.timeout" value="600000" /> 
        New Value:
        <property name="test.timeout" value="3600000" />
        <property name="test.long.timeout" value="3600000" />
        <property name="test.burn.timeout" value="3600000" /> 
```
*  To avoid timeout error for IndexSummaryManagerTest replace timeout value as shown below.
```
        e.g. Replace timeout value in IndexSummaryManagerTest.java test for testRedistributeSummaries function.
        
        Old Value:
        @Test(timeout = 10000) 
        New Value:
        @Test(timeout = 30000)
```
*  Try to run failed test case individually using below command.
```
        e.g.
        ant test -Dtest.name=LeveledCompactionStrategyTest
```
*  Apply below patch in case of TimeTypeTest and CompressorTest failed.
```
        wget https://issues.apache.org/jira/secure/attachment/12783543/11054-3.0.patch
        patch -p1 < 11054-3.0.patch	
```  
*  In case of KeyCacheCqlTest and ScrubTest failed due to key_cache size and memory issue, follow below steps.  

    * Uncomment and set the value of heap in `/<source_root>/apache-cassandra-2.2.3-src/conf/cassandra-env.sh` file.
  ```
        MAX_HEAP_SIZE="4G"
        HEAP_NEWSIZE="1G"
```  
    * Set keycache size to 12 MB in `/<source_root>/apache-cassandra-2.2.3-src/conf/cassandra.yaml` file.
  ```
        key_cache_size_in_mb: 12
```