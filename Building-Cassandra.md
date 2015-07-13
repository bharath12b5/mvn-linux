The following build instructions have been tested on **Linux on z System with IBM Java 1.7**

The following are the steps to build Cassandra 2.0.14 from source with IBM Java 1.7.
Note: This recipe is specific to IBM Java 1.7 and thus cannot be used with OpenJDK.
Please refer to the [Cassandra documentation](http://wiki.apache.org/cassandra/GettingStarted) for building with alternative JVM's.

NOTE: When following the steps below please use a standard permission user unless otherwise specified.

### Building Cassandra 2.0.14 with IBM Java 1.7

0. Download the requirements

        RHEL 7
        # use sudo yum remove <package-name> to remove openjdk if installed.
        sudo yum install java-1.7.1-ibm.s390x, libstdc++.s390x, libstdc++-devel.s390x

        SLES 12
        # use sudo zypper remove <package-name> to remove openjdk if installed.
        sudo zypper install java-1_7_1-ibm java-1_7_1-ibm-devel java-1_7_1-ibm-jdbc libstdc++-static

    The unit tests for Cassandra 2.0.14 work best with Apache Ant(TM) version 1.9.2.
    If the version is not available on the system, install the binaries using the command:

        wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.2-bin.tar.gz
        tar -xvf apache-ant-1.9.2-bin.tar.gz
        cd apache-ant-1.9.2
        export ANT_HOME=`pwd`
        cd bin
        export PATH=$PATH:`pwd`

    Cassandra also requires Snappy Java. The next step is to build Snappy Java as a jar with IBM JDK.

        mkdir snappy_z
        cd snappy_z
        wget https://github.com/linux-on-ibm-z/snappy-java/archive/develop-s390x.zip
  
        # change $JAVA_HOME to the java-1.7.1-ibm directory, e.g.,
        # example: export JAVA_PATH=/usr/lib64/jvm/java-1.7.1-ibm-1.7.1
        export JAVA_PATH=<path>
        make USE_GIT=1 GIT_REPO_URL=https://github.com/google/snappy.git GIT_SNAPPY_BRANCH=master IBM_JDK_7=1

        # the generated jar is in snappy-java/target folder - snappy-java-<version>-SNAPSHOT.jar

1. Get the Cassandra source from the Apache mirror. 2.0.14 is the current most stable release.

        wget http://mirror.its.dal.ca/apache/cassandra/2.0.14/apache-cassandra-2.0.14-src.tar.gz

2. Extract and apply patch.

        tar -xvf apache-cassandra-2.0.14-src.tar.gz
        cd apache-cassandra-2.0.14-src
 
        # apply the patch.
        wget https://raw.githubusercontent.com/karan-dh/cassandra/master/cassandra-2.0.14.patch
        patch -p2 < cassandra-2.0.14.patch
  
        # replace the original Snappy jar in the source and copy over the IBM compatible jar into the lib folder.
        rm ./lib/snappy-java-1.0.5.jar
        cp snappy-java-<version>-SNAPSHOT.jar ./lib

3. Build Cassandra

        ant

    The Cassandra jar is available under the bin folder.

4. Run the unit test.
  
        cd apache-cassandra-2.0.14-src
        ant test