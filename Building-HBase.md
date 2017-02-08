# Building HBase

The instructions provided below specify the steps to build HBase version 1.2.4 on IBM z Systems for following distributions:

* RHEL (6.8, 7.1, 7.2, 7.3)
* SLES (11 SP4, 12, 12 SP1, 12 SP2)
* Ubuntu (16.04, 16.10)

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Prerequisites(For IBM SDK only):

  * HBase needs Hadoop jar while building. To build HBase with IBM SDK, Hadoop (2.5.1) should be built first with IBM SDK. If not compiled then while starting the HBase shell, it will throw an error message saying `Unable to find JAAS classes:com.ibm.security.auth.LinuxPrincipal`.


##Step 1: Building and Installing HBase

####1.1) Install dependencies

* RHEL 6.8 
  * With IBM SDK

    ```bash
    sudo yum install -y git wget tar make gcc java-1.8.0-ibm java-1.8.0-ibm-devel ant ant-junit.s390x ant-nodeps.s390x
    ```
* RHEL (7.1, 7.2, 7.3)
  * With IBM SDK
    ```bash
    sudo yum install -y git wget tar make gcc maven java-1.8.0-ibm java-1.8.0-ibm-devel ant ant-junit.noarch
    ```

  * With OpenJDK
    ```bash
    sudo yum install -y git wget tar make gcc maven java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x ant ant-junit.noarch
    ```

* SLES (11 SP4, 12)
  * With IBM SDK
    ```bash
    sudo zypper install git wget tar make gcc ant ant-junit ant-nodeps awk net-tools
    ```

    Download [IBM SDK 8](https://developer.ibm.com/javasdk/downloads/) binary and follow the instructions mentioned in the link.  

* SLES (12 SP1, 12 SP2)
  * With IBM SDK
    ```bash
    sudo zypper install git wget tar make gcc java-1_8_0-ibm java-1_8_0-ibm-devel ant ant-junit ant-nodeps net-tools
    ```

  * With OpenJDK
    ```bash
    sudo zypper install git wget tar make gcc java-1_8_0-openjdk java-1_8_0-openjdk-devel ant ant-junit ant-nodeps net-tools
    ```

* Ubuntu (16.04, 16.10)	
  * With IBM SDK   
    ```bash
    sudo apt-get update
    sudo apt-get install -y git wget maven tar make gcc ant
    ```

    Download [IBM SDK 8](https://developer.ibm.com/javasdk/downloads/) binary and follow the instructions mentioned in the link. 

  * With OpenJDK
    ```bash
    sudo apt-get update
    sudo apt-get install -y git openjdk-8-jdk wget maven tar make gcc ant
    ```

####1.2) Download and install maven (for SLES and RHEL 6.8)
  ```bash
  cd /<source_root>/
  wget https://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
  tar zxf apache-maven-3.3.9-bin.tar.gz
  ```

####1.3) Set environment variable

  ```bash
  export JAVA_HOME=<path to java>
  export PATH=$JAVA_HOME/bin:$PATH 
  export PATH=$JAVA_HOME/bin:/<source_root>/apache-maven-3.3.9/bin:$PATH #(for SLES and RHEL 6.8)
  export HADOOP_USER_NAME="hadoop" #(Only with IBM SDK)
  export ANT_HOME=/usr/share/ant/ #(for SLES11 SP4 and RHEL 6.8)
  ```

####1.4) Get the HBase source

  ```bash
  cd /<source_root>/
  git clone https://github.com/apache/hbase.git
  cd hbase
  git checkout rel/1.2.4
  ```

####1.5) Build HBase

  ```bash
  mvn package -DskipTests
  ```

####1.6) Run test cases(Optional)  

  ```bash
  mvn test -fn
  ```

  _**Note:** Following test failures are observed on s390x_
  
  * Tests which are failing with OpenJDK consistently 
    * org.apache.hadoop.hbase.filter.TestFuzzyRowFilter: This test fails with OpenJDK 8 because function `java.nio.Bits.unaligned()` doesn't return true on s390x. This issue is been tracked [here](http://mail.openjdk.java.net/pipermail/jdk8u-dev/2017-February/006396.html).  

    * org.apache.hadoop.hbase.procedure2.store.TestProcedureStoreTracker: This test fails with OpenJDK 8 due to time out. Increasing `surefire.timeout` value in pom.xml to around 9000, resolves this test case.
       ```diff
--- a/pom.xml
+++ b/pom.xml
@@ -1254,7 +1254,7 @@
       <test.output.tofile>true</test.output.tofile>
-  <surefire.timeout>900</surefire.timeout>
+  <surefire.timeout>9000</surefire.timeout>
       <test.exclude.pattern></test.exclude.pattern>
       ```

  * Tests which are failing with OpenJDK 8 and IBM SDK intermittently 

      _**Note:** The following is a list of tests which are passed after either increasing the timeout value or rerunning individual test._ 

    * org.apache.hadoop.hbase.client.TestAsyncProcess
    * org.apache.hadoop.hbase.TestChoreService
    * org.apache.hadoop.hbase.util.TestWeakObjectPool
    * org.apache.hadoop.hbase.TestTimeout
    * org.apache.hadoop.hbase.io.hfile.TestHFileEncryption
    * org.apache.hadoop.hbase.master.balancer.TestRegionLocationFinder
    * org.apache.hadoop.hbase.regionserver.TestScanner
    * org.apache.hadoop.hbase.ipc.TestAsyncIPC
    * org.apache.hadoop.hbase.security.TestSecureRPC
    * org.apache.hadoop.hbase.TestJMXListener
    * org.apache.hadoop.hbase.coprocessor.TestRegionServerObserver
    * org.apache.hadoop.hbase.quotas.TestRateLimiter
    * org.apache.hadoop.hbase.regionserver.TestHeapMemoryManage
    * org.apache.hadoop.hbase.thrift.TestThriftHttpServer
    * org.apache.hadoop.hbase.rest.TestDeleteRow
    * org.apache.hadoop.hbase.rest.TestGetAndPutResource

##Step 2: Start HBase shell

####2.1) Get `jffi` source code 

_**Note:** HBase needs a native library (libjffi-1.0.so: java foreign language interface)_

  ```bash
  cd /<source_root>/
  wget https://github.com/jnr/jffi/archive/1.0.0.tar.gz
  tar -xvf 1.0.0.tar.gz
  tar -xvf 1.0.0 #(for SLES11 SP4)
  cd jffi-1.0.0/
  ```

####2.2) Edit following files
* `/<source_root>/jffi-1.0.0/jni/GNUmakefile`
  ```diff
--- a/jni/GNUmakefile
+++ b/jni/GNUmakefile
@@ -68,7 +68,7 @@ WERROR = -Werror
 ifneq ($(OS),darwin)
   WFLAGS += -Wundef $(WERROR)
 endif
-WFLAGS += -W -Wall -Wno-unused -Wno-parentheses -Wundef
+WFLAGS += -W -Wall -Wno-unused -Wno-parentheses -Wundef -Wno-unused-parameter
 PICFLAGS = -fPIC
 SOFLAGS = # Filled in for each OS specifically

@@ -170,7 +170,7 @@ ifeq ($(OS), darwin)
 endif

 ifeq ($(OS), linux)
-  SOFLAGS = -shared -mimpure-text -static-libgcc -Wl,-soname,$(@F) -Wl,-O1
+  SOFLAGS = -shared -static-libgcc -Wl,-soname,$(@F) -Wl,-O1
   CFLAGS += -pthread
 endif

  ```

* `/<source_root>/jffi-1.0.0/libtest/GNUmakefile` 

  ```diff
--- a/libtest/GNUmakefile
+++ b/libtest/GNUmakefile
@@ -45,9 +45,9 @@ TEST_OBJS := $(patsubst $(SRC_DIR)/%.c, $(TEST_BUILD_DIR)/%.o, $(TEST_SRCS))
 #   http://weblogs.java.net/blog/kellyohair/archive/2006/01/compilation_of_1.html
 JFLAGS = -fno-omit-frame-pointer -fno-strict-aliasing
 OFLAGS = -O2 $(JFLAGS)
-WFLAGS = -W -Werror -Wall -Wno-unused -Wno-parentheses
+WFLAGS = -W -Werror -Wall -Wno-unused -Wno-parentheses -Wno-unused-parameter
 PICFLAGS = -fPIC
-SOFLAGS = -shared -mimpure-text -Wl,-O1
+SOFLAGS = -shared -Wl,-O1
 LDFLAGS += $(SOFLAGS)

 IFLAGS = -I"$(BUILD_DIR)"

  ```

* `/<source_root>/jffi-1.0.0/custom-build.xml` (for SLES11 SP4 and RHEL 6.8)

  ```diff
--- a/custom-build.xml
+++ b/custom-build.xml
    <target name="-generate-version" depends="init,-init-vars,-generate-version-source">
-       <javac target="1.5" destdir="${build.classes.dir}" srcdir="${build.dir}/java"/>
+       <javac target="1.8" destdir="${build.classes.dir}" srcdir="${build.dir}/java"/>
    </target>
  ```

####2.3) Build with ant 

  ```bash
  ant
  ```

  _**Note:** There are some test case failures. The library file is created in `/<source_root>/jffi-1.0.0/build/jni/libjffi-1.0.so`_ 

####2.4) Add `libjffi-1.0.so` to `jruby-complete-1.6.8.jar` for s390x 

  ```bash
  mkdir /<source_root>/jar_tmp
  cp ~/.m2/repository/org/jruby/jruby-complete/1.6.8/jruby-complete-1.6.8.jar /<source_root>/jar_tmp
  cd /<source_root>/jar_tmp
  jar xf jruby-complete-1.6.8.jar
  mkdir jni/s390x-Linux
  cp /<source_root>/jffi-1.0.0/build/jni/libjffi-1.0.so jni/s390x-Linux
  jar uf jruby-complete-1.6.8.jar jni/s390x-Linux/libjffi-1.0.so
  mv jruby-complete-1.6.8.jar ~/.m2/repository/org/jruby/jruby-complete/1.6.8/jruby-complete-1.6.8.jar
  ```

####2.5) Start HBase shell
   
  ```bash
  cd /<source_root>/hbase
  bin/start-hbase.sh
  bin/hbase shell
  ```

##References:
https://hbase.apache.org/   
https://github.com/apache/hbase
