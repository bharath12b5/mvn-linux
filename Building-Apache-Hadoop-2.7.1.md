Building Apache Hadoop 2.7.1

The **Apache Hadoop** software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

Hadoop requires Google Protobuf 2.5.0, please see [[Building Google Protobuf 2.5.0]] for more information.

1. Download Hadoop source code:

    ```shell
    wget https://github.com/apache/hadoop/archive/release-2.7.1.tar.gz
    tar zxvf release-2.7.1.tar.gz
    cd hadoop-release-2.7.1
    ```

2. Edit file `./hadoop-common-project/hadoop-common/src/main/java/org/apache/hadoop/security/UserGroupInformation.java`, navigate to `line 360` and change

    ```java
    System.getProperty("os.arch").contains("64");
    ```

    to

    ```java
    System.getProperty("os.arch").contains("64") ||　System.getProperty("os.arch").contains("s390x");
    ```
    
    This fix is only needed for version below 2.8, for more information about this fix, please see https://issues.apache.org/jira/browse/HADOOP-12081

3. Run Maven to Create binary distribution with native code:

        mvn package -Pdist,native -DskipTests -Dtar

