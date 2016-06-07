### Building Apache Cassandra

Apache Cassandra is a scalable and fault-tolerant distributed NoSQL database with support for column indexes and denormalized collections. The stable release of Cassandra 2.1.13 has been built and tested on Linux on z Systems.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Building Apache Cassandra 2.1.13 with OpenJDK

The following build instructions have been tested with Apache Cassandra 2.1.13 on Linux on Z Systems with OpenJDK 1.8.

### Step 1: Install the dependencies

* RHEL 7.2:
```
sudo yum install -y git which java-1.8.0-openjdk-devel.s390x libstdc++-devel gcc-c++ make automake autoconf libtool libstdc++-static tar wget patch words
```
* SLES12 (SP1):
```
sudo zypper install -y java-1_8_0-openjdk-devel git which gcc-c++ make automake autoconf libtool libstdc++-devel tar wget patch words
```
### Step 2: Set environment variables
```    
unset JAVA_TOOL_OPTIONS
export LANG="en_US.UTF-8"
export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
```        
### Step 3: Install Ant
```
cd /<source_root>/
wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.2-bin.tar.gz
tar -xvf apache-ant-1.9.2-bin.tar.gz
cd apache-ant-1.9.2
export ANT_HOME=`pwd`
cd bin
export PATH=$PATH:`pwd`
```
### Step 4: Install the latest version of Snappy-Java
 ```

cd /<source_root>/
git clone https://github.com/xerial/snappy-java.git
cd snappy-java
git checkout develop
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk (Only for RHEL 7.2)
export JAVA_HOME=/usr/lib64/jvm/java-1.8.0 (Only for SLES12 (SP1))
make IBM_JDK_7=1 USE_GIT=1 GIT_SNAPPY_BRANCH=master GIT_REPO_URL=https://github.com/google/snappy.git
```        
### Step 5: Build and install Apache Cassandra 2.1.13
```
cd /<source_root>/
git clone https://github.com/apache/cassandra.git
cd cassandra
git checkout cassandra-2.1.13
```        
* Replace the original Snappy-Java jar file in the lib folder with installed Snappy-Java:
```
    rm /<source_root>/cassandra/lib/snappy-java-1.0.5.2.jar
    cp /<source_root>/snappy-java/target/snappy-java-1.1.3-SNAPSHOT.jar /<source_root>/cassandra/lib/snappy-java-1.1.3.jar
```
* Build Apache Cassandra:
 
```
    ant
```  
    
### Step 6: Run unit tests

**Set IBM java at run time (Optional)**

_*Note:*_ 
_Use below commands to install IBM Java 1.8 and set it to Java Runtime Environment. Setting IBM Java for tests will be needed if user needs speed in executing the test suite. User may get affected by lack of Oracle's GC functionality in IBM Java. Test case results shown below are executed using openjdk 1.8, results may vary if using IBM java_.

* RHEL 7.2:
```
    yum install java-1.8.0-ibm-devel.s390x
    export JAVA_HOME=/usr/lib/jvm/java-1.8.0-ibm-1.8.0.3.0-1jpp.1.el7.s390x/
    update-alternatives --config java    (Enter number to select IBM Java 1.8) 
```
* SLES 12 (SP1):
```
    zypper install java-1_8_0-ibm-devel
    export JAVA_HOME=/usr/lib64/jvm/java-1.8.0-ibm-1.8.0/
    update-alternatives --config java   (Enter number to select IBM Java 1.8)
```

**Run Cassandra test suite**

```
cd /<source_root>/cassandra/
ant test
```        
_**Note:**_ 
_Ignore `LegacySSTableTest` failure as it is not related to system Z, also ScrubTest failure is fixed in Apache Cassandra 2.2.6,3.0 versions. You may try to increase the value of timeout, apply patch as shown below to avoid test case failures._
    
*  Try to run failed test case individually using below command.
```
    e.g.
    ant test -Dtest.name=LeveledCompactionStrategyTest
```
*  Replace time out value in build.xml file as shown below
```
    Older Value:
    <property name="test.timeout" value="240000" />
    <property name="test.long.timeout" value="600000" />
    <property name="test.burn.timeout" value="600000" /> 
    New Value:
    <property name="test.timeout" value="1500000" />
    <property name="test.long.timeout" value="1500000" />
    <property name="test.burn.timeout" value="1500000" /> 
```
*  Apply below patch in case of NativeCellTest failed.
```
    wget https://issues.apache.org/jira/secure/attachment/12789190/11214-cassandra-3.0.txt        
    patch -p1 < 11214-cassandra-3.0.txt	
```
*  In case of KeyCacheCqlTest failed due to key_cache size and memory issue, add below content in `/<source_root>/cassandra/test/conf/cassandra.yaml` file.
```
    key_cache_size_in_mb: 12
```

### Step 7: Start Cassandra Server

_**Note:**_ To execute Cassandra binary with IBM Java (optional)  
Comment `JVM_OPTS="$JVM_OPTS -Xloggc:${CASSANDRA_HOME}/logs/gc.log"` line in `/<source_root>/casssandra/conf/cassandra-env.sh` file.

```
nohup /<source_root>/bin/cassandra -f &
```
