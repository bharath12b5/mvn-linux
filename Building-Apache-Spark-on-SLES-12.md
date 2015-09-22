1. As root, install Apache Maven:

        sudo zypper install maven

2. Obtain the Spark source code:

        git clone https://github.com/linux-on-ibm-z/spark
        cd spark
        git checkout branch-1.5

3. Download and install IBM JDK 8 from [developerWorks](http://www.ibm.com/developerworks/java/jdk/linux/download.html). Choose the "64-bit System z" version. After installing the JDK, set the `JAVA_HOME` environment variable to point its installation location:

        export JAVA_HOME=${PATH TO YOUR JDK INSTALLATION} # e.g. /usr/lib64/jvm/java-1.8.0-ibm
        export PATH=$JAVA_HOME/bin:$PATH

4. Set the `SPARK_HOME` directory:

        export SPARK_HOME=$HOME/spark

5. Invoke `mvn` to start the building process (this takes approximately 6 hours):

        mvn -Pyarn -Phadoop-2.6 -Phive -Phive-thriftserver -DskipTests package

6. Apply fixes for Linux on z:

        # We need to copy the fixed UserGroupInformation.class into the Spark assembly JAR (replacing the original)
        rm -rf org
        mkdir -p org/apache/hadoop/security

        wget anonymous@javaserv.hursley.ibm.com:/defects/hadoop-jaas-fix/*.class -P org/apache/hadoop/security

        jar uf assembly/target/scala-2.10/spark-assembly-1.5.0-SNAPSHOT-hadoop2.6.0.jar org/apache/hadoop/security/*.class
        # Update this too--it is on the classpath for tests

        jar uf $HOME/.m2/repository/org/apache/hadoop/hadoop-common/2.6.0/hadoop-common-2.6.0.jar org/apache/hadoop/security/*.class

        # Fix applied now, grab snappy jar too

        wget anonymous@javaserv.hursley.ibm.com:/defects/snappy-s390x-fix/snappy-java-1.1.1.7-ibmjdk7-s390x.jar -P $JAVA_HOME/jre/lib

        # Find line with bootpath and add :snappy-java-1.1.1.7-ibmjdk7-s390x.jar
        sed '/bootpath=/s/$/:snappy-java-1.1.1.7-ibmjdk7-s390x.jar/' $JAVA_HOME/jre/lib/classlib.properties | tee $JAVA_HOME/jre/lib/classlib.properties

7. **(Optional)** Build jblas and replace the original JAR file in `$HOME/.m2/repository/org/jblas/jblas/1.2.4/jblas-1.2.4.jar`. See [[Building Jblas]] for more information.