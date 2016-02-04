# Building Apache Flume

The instructions describe how to build the latest development version of [Apache Flume](https://flume.apache.org/) for Linux on IBM z Systems running SLES 12 or RHEL 7.

_**GENERAL NOTE:** When following the steps below please use a standard permission user unless otherwise specified. 90MB of local disk space is required._

Building Apache Flume for Linux on z Systems is a two-stage process:

1. Obtain the Flume dependencies including OpenJDK 1.6.0 or above, Maven and Google Protocol Buffer.
2. Build Flume from source code.

For more generic information on how the Flume Data flow works take a look at this [Flume Architecture](https://flume.apache.org/FlumeUserGuide.html).

## Obtaining build dependencies

Dependencies include OpenJDK, Maven and Google Protocol Buffer. This guide has been tested on SLES 12 and RHEL 7.x.

1. Install Java1.7.0 open JDK and other dependencies:

  For RHEL 7
    ```
  yum install java-1.7.0-openjdk-devel.s390x ant tar wget
    ```
  For SLES 12
    ```
  zypper install java-1_7_0-openjdk ant tar wget
    ```

1. Create a temporary working directory:

    ```
mkdir /<source_root>
cd /<source_root>/
export WORK_DIR=`pwd`
    ```
1. Path Configuration:

  For SLES12:
  ```
  export JAVA_HOME=/usr/lib64/jvm/java-1.7.0
  export M2_HOME=$WORK_DIR/maven
  export PATH=$JAVA_HOME/bin:$PATH:$M2_HOME/bin
  ```
  For RHEL7:
   ```
export JAVA_HOME=/usr/lib/jvm/java-1.7.0 export
M2_HOME=/usr/mvn/maven
export PATH=$JAVA_HOME/bin:$PATH:$M2_HOME/bin
   ```

1. Build Maven:

    ```
cd $WORK_DIR/apache-maven-3.2.5
ant
    ```

1. Build ProtoBuf:

  The Google Protobuf code can be built for Linux on z Systems running RHEL 7 and SLES 12 by following [the instructions](https://github.com/linux-on-ibm-z/docs/wiki/Building-ProtoBuf).

## Building Flume

1. Checkout the Source.

    ```
   git clone https://github.com/apache/flume.git flume
   cd flume
    ```

1. Configure the Maven option setting
Apache Flume build requires more memory than the default configuration.

    ```
export MAVEN_OPTS="-Xms512m -Xmx1024m -XX:PermSize=256m -XX:MaxPermSize=512m"
    ```

1. Build the code
  ```
# Build the code and run the tests (note: use mvn install, not mvn package, since we deploy Jenkins SNAPSHOT jars daily, and Flume is a multi-module project)
   mvn install
# or build the code without running the tests
   mvn install -DskipTests
  ```
This produces the ready-to-run snapshot packages in flume-ng-dist/target/apache-flume-ng-dist-1.4.0-SNAPSHOT-bin.tar.gz.

## Verify the build

1. Create example.conf to set up a single node data streaming server
  ```
   cd $FLUME_HOME/flume-ng-dist/target//apache-flume-1.7.0-SNAPSHOT-bin/apache-flume-1.7.0-SNAPSHOT-bin
   vim example.conf
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
1. Run Flume NG
  ```
   bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=INFO,console
  ```

1. Verify
   Open another session and input the following command:
  ```
   telnet 127.0.0.1 44444
  ```
   It will start an interactive window. Typing "Hello World" in the interactive window will stream the message in the server console.

## Reference
Flume User Guide: https://flume.apache.org/FlumeUserGuide.html

Apache in Confluence Community: https://cwiki.apache.org/confluence/display/FLUME/Getting+Started
 