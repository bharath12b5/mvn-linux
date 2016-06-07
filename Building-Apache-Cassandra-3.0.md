### Building Apache Cassandra

Apache Cassandra is a scalable and fault-tolerant distributed NoSQL database with support for column indexes and denormalized collections. The stable release of Cassandra 3.0 has been built and tested on Linux on z Systems.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Building Apache Cassandra 3.0 with OpenJDK

The following build instructions have been tested with Apache Cassandra 3.0 on Linux on z Systems with OpenJDK 1.8.

### Step 1: Install the dependencies

* RHEL 7.2:
```
sudo yum install -y git which java-1.8.0-openjdk-devel.s390x gcc-c++ make automake autoconf libtool libstdc++-static tar wget patch words libXt-devel libX11-devel

```
* SLES12 (SP1):
```
sudo zypper in -y git which make wget tar zip unzip words gcc-c++ patch libtool automake autoconf ccache java-1_8_0-openjdk-devel xorg-x11-proto-devel xorg-x11-devel alsa-devel cups-devel libffi48-devel libstdc++6-locale glibc-locale libstdc++-devel libXt-devel libX11-devel 
 
```

* Ubuntu 16.04:
```
sudo apt-get install git tar g++ make automake autoconf libtool  wget patch libx11-dev libxt-dev openjdk-8-jre openjdk-8-jdk pkg-config texinfo locales-all
 
```

### Step 2: Set environment variables
```    
unset JAVA_TOOL_OPTIONS
export LANG="en_US.UTF-8"
export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
export ANT_OPTS="-Xms4G -Xmx4G"
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
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk (Only for RHEL7.1)
export JAVA_HOME=/usr/lib64/jvm/java-1.8.0-openjdk-1.8.0 (Only for SLES12-SP1)
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x (Only for Ubuntu 16.04)
make IBM_JDK_7=1 USE_GIT=1 GIT_SNAPPY_BRANCH=master GIT_REPO_URL=https://github.com/google/snappy.git
```        
### Step 5: Build Apache Cassandra 3.0
```
cd /<source_root>/
git clone https://github.com/apache/cassandra.git
cd cassandra
git checkout cassandra-3.0
```        
* Replace the original Snappy-Java jar file in the lib folder with installed Snappy-Java:
```
    rm /<source_root>/cassandra/lib/snappy-java-1.1.1.7.jar
    cp /<source_root>/snappy-java/target/snappy-java-1.1.3-SNAPSHOT.jar /<source_root>/cassandra/lib/snappy-java-1.1.3.jar
```    
* Build JNA:
```
    cd /<source_root>/
    git clone https://github.com/java-native-access/jna.git
    cd jna
    ant
    cp /<source_root>/jna/build/jna.jar /<source_root>/cassandra/lib/jna.jar
    rm /<source_root>/cassandra/lib/jna-4.0.0.jar

```
* Build Apache Cassandra:
```
    cd /<source_root>/cassandra
    ant
```  

_*Note:* The Apache Cassandra jar file is available under the bin folder._

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
_Ignore `LegacySSTableTest`, `ClientModeSSTableTest` failure as these are not related to Linux on z system. Some test case may fail due to lack of enough resources. You may try to increase the value of timeout, per thread memory and key_cache as shown below to avoid test case failures._

*  Replace time out value in build.xml file as shown below to avoid timeout error for multiple test cases.
```
    Older Value:
    <property name="test.timeout" value="240000" />
    <property name="test.long.timeout" value="600000" />
    <property name="test.burn.timeout" value="600000" /> 
    New Value:
    <property name="test.timeout" value="600000" />
    <property name="test.long.timeout" value="600000" />
    <property name="test.burn.timeout" value="600000" /> 
```
  
* Set per thread memory to higher value for cql-tests to execute smoothly or to avoid test case failure due to timeout and memory error 

    `Edit the file /<source_root>/casssandra/build.xml`, and change jvmarg value (`<jvmarg value="-Xss256k"/> to <jvmarg value="-Xss512k"/>`)
    
    
*  Add below content in `/<source_root>/cassandra/test/conf/cassandra.yaml` to change the key cache size as per requirement
```
    key_cache_size_in_mb: 12
```

### Step 7: Start Cassandra Server

_**Note:**_ To execute Cassandra binary with IBM Java (optional)  
Comment `JVM_OPTS="$JVM_OPTS -Xloggc:${CASSANDRA_HOME}/logs/gc.log"` line in `/<source_root>/casssandra/conf/cassandra-env.sh` file.

```
nohup /<source_root>/bin/cassandra -f &
```
