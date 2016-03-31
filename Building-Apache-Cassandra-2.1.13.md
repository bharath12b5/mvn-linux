### Building Apache Cassandra 2.1.13

Apache Cassandra is a scalable and fault-tolerant distributed NoSQL database with support for column indexes and denormalized collections. The stable release of Cassandra 2.1.13 has been built and tested on Linux on z Systems.

_**General Notes:**_ 	 
_When following the steps below please use a standard permission user unless otherwise specified._

### Building Apache Cassandra 2.1.13 with OpenJDK

The following build instructions have been tested with Apache Cassandra 2.1.13 on Linux on Z Systems with Java OpenJDK.

### Step 1: Install the dependencies:

* RHEL 7:
```
sudo yum install -y git which java-1.8.0-openjdk-devel.s390x libstdc++-devel gcc-c++ make automake autoconf libtool libstdc++-static tar wget patch words
```
* SLES12:
```
sudo zypper install -y java-1_7_0-openjdk-devel git which gcc-c++ make automake autoconf libtool libstdc++-devel tar wget patch words
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
### Step 5: Build Apache Cassandra:
```
cd /<source_root>/
git clone https://github.com/apache/cassandra.git
cd cassandra
git checkout cassandra-2.1.13
```        
* Replace the original Snappy-Java jar file in the lib folder with installed Snappy-Java:
```
rm /cassandra/lib/snappy-java-1.0.5.2.jar
cp /snappy-java/target/snappy-java-1.1.3-SNAPSHOT.jar /cassandra/lib/snappy-java-1.1.3.jar
```
* Run ant command to create Apache Cassandra jar file in bin folder: 
```
ant
```  
    
### Step 6: (Optional) Set IBM java at run time:

_*Note:*_ 
_Use below commands to install IBM Java 1.7 and set it to Java Runtime Environment. Test case results shown in step 7 are executed using openjdk , results may vary if using IBM java_.

* RHEL 7:
```
yum install java-1.7.1-ibm-devel.s390x
export JAVA_HOME=/usr/lib/jvm/java-1.7.1-ibm-1.7.1.3.10-1jpp.1.el7_1.s390x
update-alternatives --config java    (Enter number to select IBM Java 1.7) 
```
* SLES 12:
```
zypper install java-1_7_1-ibm-devel java-1_7_1-ibm-jdbc java-1_7_1-ibm
export JAVA_HOME=/usr/lib64/jvm/java-1.7.1-ibm
update-alternatives --config java   (Enter number to select IBM Java 1.7)
```

### Step 7: Run the unit tests:

Run the Cassandra test suite         
```
ant test
```        
_**Note:**_ 
_At present few test cases are failing, failed test cases are reproduced on x86 except ScrubTest and LegacySSTableTest. Try below changes related to system Z to avoid test case failures due to timeout._
    
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
*  Replace timeout value in particular test case
```
        e.g. In IndexSummaryManagerTest.java test for function testRedistributeSummaries replace
        
        Old Value:
        @Test(timeout = 10000) 
        New Value:
        @Test(timeout = 30000)
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