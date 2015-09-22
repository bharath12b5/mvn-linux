1. As root, install Apache Maven:

        sudo zypper install maven

2. Obtain the Spark source code (access to Hursley's gitlab is required; alternatively, download the tar.gz attached to this work item):

        git clone https://github.com/linux-on-ibm-z/spark
        git checkout branch-1.5

3. Install IBM JDK8, see instructions here: http://www.ibm.com/developerworks/java/jdk/linux/download.html (select `64-bit System z` version). Then set the JAVA_HOME path:

        export JAVA_HOME=${PATH TO YOUR JDK INSTALLATION} # i.e. /usr/lib64/jvm/jre-1.8.0-openjdk
        export PATH=$JAVA_HOME/bin:$PATH

4. Set the `SPARK_HOME` directory:

        export SPARK_HOME=$HOME/PROD_SPARK/spark-testing

5. Invoke `mvn` to start the building process (this takes approximately 6 hours):

        mvn -Pyarn -Phadoop-2.6 -Phive -Phive-thriftserver -DskipTests package

6. Apply fixes for Linux on z:

        # We need to copy the fixed UserGroupInformation.class into the spark assembly jar (replacing what's in there atm)
        rm -rf org
        mkdir -p org/apache/hadoop/security

        wget anonymous@javaserv.hursley.ibm.com:/defects/hadoop-jaas-fix/*.class -P org/apache/hadoop/security

        jar uf assembly/target/scala-2.10/spark-assembly-1.5.0-SNAPSHOT-hadoop2.6.0.jar org/apache/hadoop/security/*.class
        # So update this too - it's on the classpath for tests

        jar uf $HOME/.m2/repository/org/apache/hadoop/hadoop-common/2.6.0/hadoop-common-2.6.0.jar org/apache/hadoop/security/*.class

        # Fix applied now, grab snappy jar too

        wget anonymous@javaserv.hursley.ibm.com:/defects/snappy-s390x-fix/snappy-java-1.1.1.7-ibmjdk7-s390x.jar -P $JAVA_HOME/jre/lib

        # Find line with bootpath and add :snappy-java-1.1.1.7-ibmjdk7-s390x.jar
        sed '/bootpath=/s/$/:snappy-java-1.1.1.7-ibmjdk7-s390x.jar/' $JAVA_HOME/jre/lib/classlib.properties | tee $JAVA_HOME/jre/lib/classlib.properties

7. OPTIONAL: Build Jblas and apply the patched jar in `$HOME/.m2/repository/org/jblas/jblas/1.2.4/jblas-1.2.4.jar`. See [[Build Jblas]] for more information.