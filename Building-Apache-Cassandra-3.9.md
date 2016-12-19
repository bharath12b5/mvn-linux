# Building Apache Cassandra

The instructions provided below specify the steps to build Apache Cassandra version 3.9 on IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10:

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##Step 1: Building and installing Apache Cassandra

####1.1) Install dependencies
* RHEL 7.1/7.2/7.3
  ```bash
  sudo yum install git which java-1.8.0-openjdk-devel.s390x gcc-c++ make automake autoconf libtool libstdc++-static tar wget patch words libXt-devel libX11-devel texinfo
  ```
  
* SLES 12-SP1/12-SP2
  ```bash
  sudo zypper install git which make wget tar zip unzip words gcc-c++ patch libtool automake autoconf ccache java-1_8_0-openjdk-devel xorg-x11-proto-devel xorg-x11-devel alsa-devel cups-devel libffi48-devel libstdc++6-locale glibc-locale libstdc++-devel libXt-devel libX11-devel texinfo 
  ```

* Ubuntu 16.04/16.10
  ```bash
  sudo apt-get update
  sudo apt-get install git tar g++ make automake autoconf libtool  wget patch libx11-dev libxt-dev openjdk-8-jre openjdk-8-jdk pkg-config texinfo locales-all 
  ```

####1.2) Install Ant
  ```bash
  cd /<source_root>/
  wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.4-bin.tar.gz
  tar -xvf apache-ant-1.9.4-bin.tar.gz
  ```

####1.3) Set environment variables
  ```bash
  unset JAVA_TOOL_OPTIONS
  export LANG="en_US.UTF-8"
  export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
  export JAVA_HOME=/usr/lib/jvm/java #(for RHEL)
  export JAVA_HOME=/usr/lib64/jvm/java #(for SLES)
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x #(for Ubuntu)
  export ANT_OPTS="-Xms4G -Xmx4G"
  export ANT_HOME=/<source_root>/apache-ant-1.9.4
  export PATH=$PATH:$ANT_HOME/bin
  ```

####1.4) Download source code
  ```bash
  cd /<source_root>/
  git clone https://github.com/apache/cassandra.git
  cd cassandra
  git checkout cassandra-3.9
  ```

####1.5) Build Apache Cassandra:
  ```bash
  cd /<source_root>/cassandra
  ant
  ``` 

##Step 2: Testing(Optional)
####2.1) Install jar files required for testing on IBM z Systems
* Build Snappy-Java
  ```bash
  cd /<source_root>/
  git clone https://github.com/xerial/snappy-java.git
  cd snappy-java
  git checkout develop
  make IBM_JDK_7=1 USE_GIT=1 GIT_SNAPPY_BRANCH=master GIT_REPO_URL=https://github.com/google/snappy.git
  ```        

* Build JNA
  ```bash
  cd /<source_root>/
  git clone https://github.com/java-native-access/jna.git
  cd jna
  ant
  ```

* Replace the jar files
  ```bash
  rm /<source_root>/cassandra/lib/snappy-java-1.1.1.7.jar
  rm /<source_root>/cassandra/lib/jna-4.0.0.jar
  cp /<source_root>/snappy-java/target/snappy-java-1.1.3-SNAPSHOT.jar /<source_root>/cassandra/lib/snappy-java-1.1.3.jar
  cp /<source_root>/jna/build/jna.jar /<source_root>/cassandra/lib/jna.jar
  ```

####2.2) Run test suit
  ```bash
  cd /<source_root>/cassandra/
  ant test
  ```        
_**Note:** You may resolve some testcase failure by applying the following changes:_ 
* Increase timeout in `/<source_root>/casssandra/build.xml` as shown below for timeout related testcase failure
  ```diff
  @@ -97,7 +97,7 @@
       <property name="maven-repository-url" value="https://repository.apache.org/content/repositories/snapshots"/>
       <property name="maven-repository-id" value="apache.snapshots.https"/>
  
   -    <property name="test.timeout" value="240000" />
   +    <property name="test.timeout" value="600000" />
         <property name="test.long.timeout" value="600000" />
         <property name="test.burn.timeout" value="600000" />
  
  ```

* Set per thread memory to higher value for cql-tests in `/<source_root>/casssandra/build.xml`
  ```diff
  @@ -1497,7 +1497,7 @@
           <jvmarg value="-Djava.awt.headless=true"/>
           <jvmarg value="-javaagent:${basedir}/lib/jamm-0.3.0.jar" />
           <jvmarg value="-ea"/>
   -        <jvmarg value="-Xss256k"/>
   +        <jvmarg value="-Xss512k"/>
             <jvmarg value="-Dcassandra.memtable_row_overhead_computation_step=100"/>
             <jvmarg value="-Dcassandra.test.use_prepared=${cassandra.test.use_prepared}"/>
             <jvmarg value="-Dcassandra.skip_sync=true" />
  @@ -1539,7 +1539,7 @@
           <jvmarg value="-Djava.awt.headless=true"/>
           <jvmarg value="-javaagent:${basedir}/lib/jamm-0.3.0.jar" />
           <jvmarg value="-ea"/>
   -        <jvmarg value="-Xss256k"/>
   +        <jvmarg value="-Xss512k"/>
             <jvmarg value="-Dcassandra.test.use_prepared=${cassandra.test.use_prepared}"/>
             <jvmarg value="-Dcassandra.memtable_row_overhead_computation_step=100"/>
             <jvmarg value="-Dcassandra.skip_sync=true" />
  ```
    
*  Change the key cache size as per requirement in `/<source_root>/cassandra/test/conf/cassandra.yaml`
  ```diff
  @@ -44,3 +44,4 @@ row_cache_class_name: org.apache.cassandra.cache.OHCProvider
    row_cache_size_in_mb: 16
    enable_user_defined_functions: true
    enable_scripted_user_defined_functions: true
  + key_cache_size_in_mb: 12
  ```

##Step 3: Start Cassandra Server(Optional)
```bash
./<source_root>/cassandra/bin/cassandra -f
```
_**Note:** To use complete functionality of Apache Cassandra, build and copy jar files to lib directory as mentioned above in step 2.1 if not done earlier._

##References:
http://cassandra.apache.org/
