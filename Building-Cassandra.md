Apache Cassandra is a scalable and fault-tolerant distributed NoSQL database with support for column indexes and denormalized collections. The stable release of Cassandra has been built and tested on Linux on z Systems.

## Building Cassandra 2.0.14 with IBM Java 1.7

The following build instructions have been tested with Cassandra 2.0.14 on Linux on z Systems with IBM Java 1.7.

Note: This recipe is specific to IBM Java 1.7 and thus cannot be used with OpenJDK.

Please refer to the [Cassandra documentation](http://wiki.apache.org/cassandra/GettingStarted) for building with alternative JVMs.

Note: When following the steps below please use a standard permission user unless otherwise specified.

0. Install the pre-requisite packages:

        RHEL 7
        # use sudo yum remove <package-name> to remove openjdk if installed.
        sudo yum install java-1.7.1-ibm.s390x, libstdc++.s390x, libstdc++-devel.s390x

        SLES 12
        # use sudo zypper remove <package-name> to remove openjdk if installed.
        sudo zypper install java-1_7_1-ibm java-1_7_1-ibm-devel java-1_7_1-ibm-jdbc libstdc++-static

   The unit tests for Cassandra 2.0.14 work best with Apache Ant(TM) version 1.9.2.
   If this version is not available on the system, install the binaries using these commands:

        wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.2-bin.tar.gz
        tar -xvf apache-ant-1.9.2-bin.tar.gz
        cd apache-ant-1.9.2
        export ANT_HOME=`pwd`
        cd bin
        export PATH=$PATH:`pwd`

1. Get the Cassandra 2.0.14 source code from the Apache mirror, and unpack the files.

        wget http://archive.apache.org/dist/cassandra/2.0.14/apache-cassandra-2.0.14-src.tar.gz
        tar -xvf apache-cassandra-2.0.14-src.tar.gz
        cd apache-cassandra-2.0.14-src

2. Patch the code to make it work correctly with IBM Java.

        wget https://raw.githubusercontent.com/linux-on-ibm-z/docs/master/patches/cassandra-2.0.14-ibm-java.patch
        patch -p1 < cassandra-2.0.14-ibm-java.patch
 
3. Download the latest version of [[Snappy-Java]] and replace the original Snappy-Java JAR file in the lib folder with the new one:

        wget http://search.maven.org/remotecontent?filepath=org/xerial/snappy/snappy-java/1.1.2/snappy-java-1.1.2.jar
        rm ./lib/snappy-java-1.0.5.jar
        mv snappy-java-1.1.2.jar ./lib/

4. Build Cassandra:

        ant

   The Cassandra JAR file is available under the bin folder.

4. **(Optional)** Run the unit tests:
 
        ant test
