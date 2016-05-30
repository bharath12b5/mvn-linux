<!---PACKAGE:Apache Flume--->
<!---DISTRO:SLES 12:1.6--->
<!---DISTRO:RHEL 7.1:1.6--->
<!---DISTRO:Ubuntu 16.x:1.6--->

# Building Apache Flume

The instructions describe how to build the [Apache Flume](https://flume.apache.org/) version 1.6 on IBM z Systems for SLES 12 or RHEL 7 or Ubuntu 16.04.

_**GENERAL NOTE:** When following the steps below please use a standard permission user unless otherwise specified. 90MB of local disk space is required._

Building Apache Flume for Linux on z Systems is a two-stage process

1. Obtain the Flume dependencies including IBM JDK 1.6.0 or above, Maven and Google Protocol Buffer.
2. Build Flume from source code.

For more generic information on how the Flume Data flow works take a look at this [Flume Architecture](https://flume.apache.org/FlumeUserGuide.html).

## Obtaining Build Dependencies

Dependencies include IBM JDK, Open JDK, Maven and Google Protocol Buffer. This guide has been tested on SLES 12 and RHEL 7.x and Ubuntu 16.04.

1. Install Dependencies

  For RHEL 7
    ```
	sudo yum install java-1.7.1-ibm-devel tar wget ant tar git telnet
    ```
  For SLES 12
    ```
	sudo zypper install java-1_7_1-ibm-devel  ant tar wget git telnet
    ```
  For Ubuntu 16.04
	```
	sudo apt-get update
	sudo apt-get install openjdk-8-jdk maven protobuf-compiler ant tar wget git telnet
	```
2. Create a temporary working directory

	```
	mkdir /<source_root>
	cd /<source_root>/
	export WORK_DIR=`pwd`
	```
3. Path Configuration

  For SLES12
	```
	export JAVA_HOME=/usr/lib64/jvm/java
	export M2_HOME=<source_root>/apache-maven-3.2.5/maven
	export PATH=$JAVA_HOME/bin:$PATH:$M2_HOME/bin
	```
  For RHEL7
	```
	export JAVA_HOME=/usr/lib/jvm/java
	export M2_HOME=<source_root>/apache-maven-3.2.5/maven
	export PATH=$JAVA_HOME/bin:$PATH:$M2_HOME/bin
	```
  For Ubuntu 16.04
	```
	export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
	export M2_HOME=/usr/share/maven
	export PATH=$PATH:$JAVA_HOME:$JAVA_HOME/bin:$M2_HOME/bin
	```

4. Build Maven (Only for RHEL7 and SLES12)
    ```
	wget http://apache.cs.utah.edu/maven/maven-3/3.2.5/source/apache-maven-3.2.5-src.tar.gz 
	tar -zxvf apache-maven-3.2.5-src.tar.gz 
	cd apache-maven-3.2.5
	ant
	```


5. Build ProtoBuf(Only for RHEL7 and SLES12), by following the instructions [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-ProtoBuf).

## Building Flume

1. Checkout the Source

    ```
   git clone https://github.com/apache/flume.git flume
   cd flume
   git checkout flume-1.6 
    ```

2. Configure the Maven option setting
Apache Flume build requires more memory than the default configuration.

    ```
   export MAVEN_OPTS="-Xms1024m -Xmx1024m -XX:MaxPermSize=1024m"
    ```

3. Edit the pom.xml file to change the version of snappy-java in pom.xml from 1.1.0 to 1.1.2

       ``` 
       <dependency>
         <groupId>org.xerial.snappy</groupId>
         <artifactId>snappy-java</artifactId>
         <version>1.1.2</version>
       </dependency>
       ```
4. Increase the maximum number of files that are allowed to be opened on the system to 4096 by using  "ulimit -n 4096".

5. Build the code
  ```
# Build the code and run the tests (note: use mvn install, not mvn package, since we deploy Jenkins SNAPSHOT jars daily, and Flume is a multi-module project)
   mvn install -Drat.numUnapprovedLicenses=100
# or build the code without running the tests
   mvn install -DskipTests  -Drat.numUnapprovedLicenses=100
  ```

## Verify the build

1. Create example.conf to set up a single node data streaming server
  ```
   cd /<source_root>/flume/flume-ng-dist/target/apache-flume-1.6.0-bin/apache-flume-1.6.0-bin
   vi example.conf
  ```
  Input the following content to example.conf
  ```
  # Name the components on this agent
   a1.sources = r1
   a1.sinks = k1
   a1.channels = c1

  # Describe/configure the source
     a1.sources.r1.type = netcat
     a1.sources.r1.bind = localhost
     a1.sources.r1.port = 44444

  # Describe the sink
     a1.sinks.k1.type = logger

  # Use a channel which buffers events in memory
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100

  # Bind the source and sink to the channel
   a1.sources.r1.channels = c1
   a1.sinks.k1.channel = c1
  ```
2. Run Flume NG
  ```
   bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=INFO,console
  ```

3. Verify
   Open another session and input the following command:
  ```
   telnet 127.0.0.1 44444
  ```
   It will start an interactive window. Typing "Hello World" in the interactive window will stream the message in the server console.

## Reference
Flume User Guide: https://flume.apache.org/FlumeUserGuide.html

Apache in Confluence Community: https://cwiki.apache.org/confluence/display/FLUME/Getting+Started
 
