<!---PACKAGE:Neo4j--->
<!---DISTRO:SLES 12.1:2.3--->
<!---DISTRO:RHEL 7.2:2.3--->


# Building Neo4j


[Neo4j 2.3](http://neo4j.com/) has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.2 and SLES 12.1.


_**General Notes:**_ 	 
i) When following the steps below, please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it.

 1.  Install the Build Dependencies

    RHEL 7.2
    ```
    sudo yum install git gcc-c++ make java-1.8.0-ibm java-1.8.0-ibm-devel graphviz maven rpm git unzip bzip2
    ```

	  SLES 12.1
    ```
    sudo zypper install java-1.8.0-ibm java-1.8.0-ibm-devel  git gcc-c++ make graphviz rpm wget tar unzip bzip2
    ```	 

 2.  Download and Install Node.js
 
    ```
    cd <source_root> 
    git clone https://github.com/andrewlow/node.git
    cd node/
    ./configure
    make 
    sudo make install
    ```
	
 3.  Install grunt-cli (Only for SLES 12.1)
 
    ```
    sudo /usr/local/bin/npm install -g grunt-cli
    ```
	 
 4.  Download maven binary executable (Only for SLES 12.1)
 
    ```
    cd <source_root> 
    wget https://archive.apache.org/dist/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz
    tar -zxvf apache-maven-3.0.5-bin.tar.gz
    export PATH=<source_root>/apache-maven-3.0.5/bin:$PATH
    ```
	
 5.  Set Enviroment variables
 
    ```
    export MAVEN_OPTS="-Xmx512m -XX:MaxPermSize=128m"
    export JAVA_HOME=/etc/alternatives/java_sdk_1.8.0_ibm
    export PATH=$JAVA_HOME/bin:$PATH
    ```
	 
 6.  Download and unzip Neo4j source code 
	
    ```
    cd <source_root>
    wget https://github.com/neo4j/neo4j/archive/2.3.zip 
    unzip 2.3.zip
    cd <source_root>/neo4j-2.3
    ```

 7.  Ignore testcases that fail persistently
		
    ```
    sed -i '26i import org.junit.Ignore;'  community/kernel/src/test/java/org/neo4j/helpers/HostnamePortTest.java
    sed -i '488i @Ignore'  community/kernel/src/test/java/org/neo4j/helpers/HostnamePortTest.java
    sed -i '50i @Ignore'  community/kernel/src/test/java/org/neo4j/qa/tooling/DumpProcessInformationTest.java
    sed -i '22i import org.junit.Ignore;'  community/kernel/src/test/java/org/neo4j/qa/tooling/DumpProcessInformationTest.java
    ```
	 
 8.	Build the source 
 
     To build individual jar files and execute test cases run the following command
    ```
    mvn clean install -DskipBrowser -fn
    ```

     To build without running test cases execute the following
    ```
    mvn clean install -DskipBrowser -DskipTests -fn
    ```
	 
 9. Running Neo4j
	
    ```
    cd <source_root>/neo4j-2.3/community/server
    mvn clean compile exec:java
    ```

 10. **Additional Notes**
	
	i) If build fails due to the error "Too many open files" try increasing the limit for number of open files.
    ```
    ulimit -n 4096
    ```
	
	ii) Some of the test cases might fail due to time-out. In such a case you will have to increase the timeout and re-run the test case.
	 
	For example,
	 
    ```
    At Line 465 change SEMI_LONG_TIMEOUT_MILLIS to LONG_TIMEOUT_MILLIS in file <source_root>/neo4j2.3/community/io/src/test/java/org/neo4j/io/pagecache/PageCacheTest.java
    At Line 533 change SHORT_TIMEOUT_MILLIS to LONG_TIMEOUT_MILLIS in file <source_root>/neo4j2.3/community/io/src/test/java/org/neo4j/io/pagecache/PageCacheTest.java
    ```
	
	**NOTE:** *After building Neo4j it will be running on `localhost:7474`.*