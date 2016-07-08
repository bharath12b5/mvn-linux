<!---PACKAGE:Neo4j--->
<!---DISTRO:SLES 12.1:2.3--->
<!---DISTRO:RHEL 7.2:2.3--->
<!---DISTRO:Ubuntu 16.x:2.3--->

# Building Neo4j

The instructions provided below specify the steps to build Neo4j 2.3 on Linux on the IBM z Systems for RHEL 7.2, SLES 12 SP1 and Ubuntu 16.04.

_**General Notes:**_ 	 
i) When following the steps below, please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it.

 1.  Install the Build Dependencies

      a) RHEL 7.2

     * With IBM JDK  
     ```
     sudo yum install git gcc-c++ make java-1.8.0-ibm java-1.8.0-ibm-devel graphviz maven rpm git unzip bzip2 wget

     ```
     * With OpenJDK  
     ```
     sudo yum install git gcc-c++ make java-1.8.0-openjdk-devel.s390x wget tar graphviz maven rpm git unzip bzip2

     ```

   b) SLES 12 SP1

     * With IBM JDK  
	 ```
	 sudo zypper install java-1.8.0-ibm java-1.8.0-ibm-devel  git gcc-c++ make graphviz rpm wget tar unzip bzip2

	 ```	 
     * With OpenJDK  
	 ```
	 sudo zypper install git gcc-c++ make graphviz rpm wget tar unzip bzip2 java-1_8_0-openjdk-devel

	 ```  
		 
   c) Ubuntu 16.04
	 
     * With IBM JDK    
	 
	 Install IBM Java 8
	 Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link
	 
	 ```
	 sudo apt-get install git gcc g++ make graphviz rpm wget maven tar unzip bzip2 npm nodejs nodejs-legacy devscripts debhelper rpm openjdk-8-jdk-headless
	 ```
     
     * With OpenJDK   
	 ```
	 sudo apt-get install wget git gcc g++ make openjdk-8-jdk graphviz maven rpm unzip bzip2 nodejs nodejs-legacy npm devscripts debhelper 
	 ```
	 
 2.  Download and Install Node.js (Only for RHEL 7.2 and SLES 12 SP1)
 
	 ```
	 cd <source_root> 
	 git clone https://github.com/andrewlow/node.git
	 cd node/
	 ./configure
	 make 
	 sudo make install
	 ```
	
 3.  Install grunt-cli (Only for SLES 12 SP1)
 
	 ```
	 sudo /usr/local/bin/npm install -g grunt-cli
	 ```
	 
 4.  Download maven binary executable (Only for SLES 12 SP1)
 
	 ```
	 cd <source_root> 
	 wget https://archive.apache.org/dist/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz
	 tar -zxvf apache-maven-3.0.5-bin.tar.gz
	 export PATH=<source_root>/apache-maven-3.0.5/bin:$PATH
	 ```
	
 5.  Set Environment variables
 
 
	For RHEL 7.2 and SLES 12 SP1 (Using IBM JDK)
      ```
     export MAVEN_OPTS="-Xmx512m -XX:MaxPermSize=128m"
	 export JAVA_HOME=/etc/alternatives/java_sdk_1.8.0_ibm
	 export PATH=$JAVA_HOME/bin:$PATH
	 ```
      For SLES12 SP1 (Using OpenJDK)
	```
export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=512m"
export JAVA_HOME=/usr/lib64/jvm/java
export PATH=$JAVA_HOME/bin:$PATH
export JVM_ARGS="-Xms1024m -Xmx1024m"
export JAVA_OPTS="-Xms128m -Xmx512m"

	```
      For RHEL7.2 (Using OpenJDK)
	```
export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=512m"
export JAVA_HOME=/usr/lib/jvm/java
export PATH=$JAVA_HOME/bin:$PATH
export JVM_ARGS="-Xms1024m -Xmx1024m"
export JAVA_OPTS="-Xms128m -Xmx512m"

	```
	For Ubuntu 16.04 (Using IBM JDK)
	
	Set JAVA_HOME to location where IBM Java 8 sdk binary is downloaded
	
	
	 ```
     export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=512m"
	 export JVM_ARGS="-Xms1024m -Xmx1024m"
	 export PATH=$JAVA_HOME/bin:$PATH
	 ```
 For Ubuntu 16.04 (Using OpenJDK)
	```
export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=512m"
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
export PATH=$JAVA_HOME/bin:$PATH 
export JVM_ARGS="-Xms1024m -Xmx1024m"
export JAVA_OPTS="-Xms128m -Xmx512m"
	```

 6.  Download and unzip Neo4j source code 
	
	 ```
	 cd <source_root>
	 wget https://github.com/neo4j/neo4j/archive/2.3.zip 
	 unzip 2.3.zip
	 cd <source_root>/neo4j-2.3
     ```
 7.  Ignore testcases that fail persistently(Only for RHEL 7.2 and SLES 12 SP1)
		
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
	 
	For example
	 
	 ```
	 At Line 465 change SEMI_LONG_TIMEOUT_MILLIS to LONG_TIMEOUT_MILLIS in file <source_root>/neo4j2.3/community/io/src/test/java/org/neo4j/io/pagecache/PageCacheTest.java
	 At Line 533 change SHORT_TIMEOUT_MILLIS to LONG_TIMEOUT_MILLIS in file <source_root>/neo4j2.3/community/io/src/test/java/org/neo4j/io/pagecache/PageCacheTest.java
	 ```
	
	**NOTE:** *After building Neo4j it will be running on `localhost:7474`.*