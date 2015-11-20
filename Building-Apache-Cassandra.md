### Building Apache Cassandra

Apache Cassandra is a scalable and fault-tolerant distributed NoSQL database with support for column indexes and denormalized collections. The stable release of Cassandra 2.1.11 has been built and tested on Linux on z Systems.

_**General Notes:**_ 	 
_When following the steps below please use a standard permission user unless otherwise specified._

### Building Apache Cassandra 2.1.11 with OpenJDK

The following build instructions have been tested with Apache Cassandra 2.1.11 on Linux on Z Systems with Java OpenJDK.

### Step 1: Install the dependencies:

* RHEL 7:
```
yum install -y git \
    which \
    java-1.8.0-openjdk-devel.s390x \
    libstdc++-devel \
    gcc-c++ \
    make \
    automake \
    autoconf \
    libtool \
    libstdc++-static \
    tar \
    wget \
    patch \
    words
```
* SLES12:
```
zypper install -y java-1_7_0-openjdk-devel \
    git \
    which \
    gcc-c++ \
    make \
    automake \
    autoconf \
    libtool \
    libstdc++-devel \
    tar \
    wget \
    patch \
    words
```
### Step 2: Set environment variables:
```    
unset JAVA_TOOL_OPTIONS
export LANG="en_US.UTF-8"
export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
```        
### Step 3: Install Ant:
```
cd /
wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.2-bin.tar.gz
tar -xvf apache-ant-1.9.2-bin.tar.gz
cd apache-ant-1.9.2
export ANT_HOME=`pwd`
cd bin
export PATH=$PATH:`pwd`
```
### Step 4: Install the latest version of Snappy-Java:
 ```

cd /
git clone https://github.com/xerial/snappy-java.git
cd snappy-java
git checkout develop
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk (Only for RHEL7)
export JAVA_HOME=/usr/lib64/jvm/java-1.7.0 (Only for SLES12)
make IBM_JDK_7=1 USE_GIT=1 GIT_SNAPPY_BRANCH=master GIT_REPO_URL=https://github.com/google/snappy.git
```        
### Step 5: Build and install Apache Cassandra 2.1.11:
```
cd /
wget http://www.us.apache.org/dist/cassandra/2.1.11/apache-cassandra-2.1.11-src.tar.gz
tar -xvf apache-cassandra-2.1.11-src.tar.gz
cd apache-cassandra-2.1.11-src
```        
* Replace the original Snappy-Java jar file in the lib folder with installed Snappy-Java:
```
rm /apache-cassandra-2.1.11-src/lib/snappy-java-1.0.5.2.jar
cp /snappy-java/target/snappy-java-1.1.3-SNAPSHOT.jar /apache-cassandra-2.1.11-src/lib/snappy-java-1.1.3.jar
```    
* Build Apache Cassandra:
```
ant
```
    * The Apache Cassandra jar file is available under the bin folder.
    
### Step 6: (Optional) Run the unit tests:
```
ant test
```        
_**Note:**_ At present few test cases are failing, failed test cases are reproduced on x86 except ScrubTest and LegacySSTableTest. Try below changes related to system Z to avoid test case failures due to timeout. 
    
1. Try to run failed test case individually using below command.
```
        e.g.
        ant test -Dtest.name=LeveledCompactionStrategyTest
```
2. Replace time out value in build.xml file as shown below
```
        Older Value:
        <property name="test.timeout" value="60000" />
        <property name="test.long.timeout" value="600000" />
        <property name="test.burn.timeout" value="600000" /> 
        New Value:
        <property name="test.timeout" value="1500000" />
        <property name="test.long.timeout" value="1500000" />
        <property name="test.burn.timeout" value="1500000" /> 
```
3. Replace timeout value in particular test case
```
        e.g. In IndexSummaryManagerTest.java test for function testRedistributeSummaries replace
        
        Old Value:
        @Test(timeout = 10000) 
        New Value:
        @Test(timeout = 30000)
``` 