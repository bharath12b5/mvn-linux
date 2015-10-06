Apache Spark is an open source cluster computing framework originally developed in the AMPLab at University of California, Berkeley but was later donated to the Apache Software Foundation where it remains today. In contrast to Hadoop's two-stage disk-based MapReduce paradigm, Spark's multi-stage in-memory primitives provides performance up to 100 times faster for certain applications. By allowing user programs to load data into a cluster's memory and query it repeatedly, Spark is well-suited to machine learning algorithms.

Disclaimer: Spark on Linux on z is still in beta, this implies the quality of this software is not for production use.

1. As root, install Apache Maven:

        sudo zypper install maven

2. Obtain the Spark source code:

        git clone https://github.com/linux-on-ibm-z/spark
        cd spark
        git checkout branch-1.5

3. Download and install IBM JDK 8 from [developerWorks](http://www.ibm.com/developerworks/java/jdk/linux/download.html). Choose the "64-bit System z" version. After installing the JDK, set the `JAVA_HOME` environment variable to point its installation location:

        export JAVA_HOME=${PATH TO YOUR JDK INSTALLATION} # e.g. /usr/lib64/jvm/java-1.8.0-ibm
        export PATH=$JAVA_HOME/bin:$PATH

4. **(Optional, for storing test results)** Set the `SPARK_HOME` directory:

        export SPARK_HOME=$HOME/spark

5. Invoke `mvn` to start the building process (this takes approximately 6 hours):

        mvn -Pyarn -Phadoop-2.7 -Phive -Phive-thriftserver -DskipTests package

6. Build Hadoop from source. The patches made to the Apache Hadoop release 2.7, but not yet in the Maven's repository, so we need to manually build the JAR and replace the original. See [[Building Apache Hadoop 2.7.1]] for instructions to build Apache Hadoop.

        cp ${hadoop.jar} $HOME/.m2/repository/org/apache/hadoop/hadoop-common/2.7.0/hadoop-common-2.7.0.jar 

7. Build jblas from source. See [[Building Jblas 1.2.4]] for more information. Build jblas and replace the original JAR file in `$HOME/.m2/repository/org/jblas/jblas/1.2.4/jblas-1.2.4.jar`. 