<!---PACKAGE:Elasticsearch--->
<!---DISTRO:SLES 12.x:2.4--->
<!---DISTRO:RHEL 7.x:2.4--->
<!---DISTRO:Ubuntu 16.x:2.4--->

# Building Elasticsearch

Below versions of Elasticsearch are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `1.7.3`

The instructions provided below specify the steps to build [Elasticsearch](https://www.elastic.co/products/elasticsearch) v2.4.1 on Linux on the IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10.

##### General Notes
      
* When following the steps below please use a standard permission user unless otherwise specified.

* A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it.

#### Install Build Dependencies

1. RHEL 7.1/7.2/7.3
    
    	sudo yum install java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x tar wget unzip curl which
		
	* With IBM Java 8:
	```	
	sudo yum install tar wget unzip curl which java-1.8.0-ibm-devel.s390x java-1.8.0-ibm.s390x
    ```    
2. SLES 12-SP1/12-SP2
  
    	sudo zypper install java-1_8_0-openjdk java-1_8_0-openjdk-devel tar wget unzip curl which
		
	* With IBM Java 8:
	```	
	sudo zypper install -y tar wget unzip curl which java-1_8_0-ibm java-1_8_0-ibm-devel 
	```
3. Ubuntu 16.04/16.10
  
    	sudo apt-get update
        sudo apt-get install tar wget unzip curl maven openjdk-8-jdk
		
	* Install IBM Java 8
	```
	Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.
	```	

#### Download Maven and add it to the path 

1. Download Maven (Only for RHEL and SLES)
	```
    cd /<source_root>/
    wget http://archive.apache.org/dist/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
    tar -zxvf apache-maven-3.3.3-bin.tar.gz
    ```
	
2. Set the Environment variables
   
   a. On RHEL and SLES

	 ```
		 export JAVA_HOME=/usr/lib/jvm/java-ibm (For RHEL)
		 export JAVA_HOME=/usr/lib64/jvm/java-ibm (For SLES)
         MAVEN="/<source_root>/apache-maven-3.3.3/bin"
         export PATH=$PATH:$MAVEN
         export MAVEN_OPTS="-Xms1024m -Xmx1024m -XX:MaxPermSize=1024m"
         export _JAVA_OPTIONS="-Xmx5g"  
       ```
   b. On Ubuntu 16.04/16.10

       ```
         export _JAVA_OPTIONS="-Xmx10g"  
       ``` 


3. Increase the maximum number of files that are allowed to be opened on the system
   "ulimit -n" should be greater than 32k or 64k. 32k and 64k are the suggested sizes.

	In order to increase this limit, follow the given steps.
 
	As root, edit the file /etc/security/limits.conf, add the given line
	```
    *       -       nofile      65535
    ```
    
   Also as root, run the command "ulimit -n 65535". 
	Logout and Login. Run the command ulimit -n, it should return 65535


####  Obtain required version of Elasticsearch

1. Download Elasticsearch

	```
    cd /<source_root>/
    wget https://github.com/elastic/elasticsearch/archive/v2.4.1.zip
    unzip v2.4.1.zip
    cd elasticsearch-2.4.1
	sed -i '367 s/java.runtime.version/java.version/' core/pom.xml (Only with IBM Java 8)
    ```
	
#### Build the package

1. Run following commands to build the package 

     a. without the tests
	
	```
	mvn clean package -DskipTests
	```	
   b. with local tests
  
       ```
	mvn clean package -Des.node.mode=local
	```	

   c. with network tests
  
       ```
	mvn clean package -Des.node.mode=network
	```	

	_**Note:**_ Elasticsearch uses randomized testing and hence many tests might fail unpredictably. If the -DskipTests succeeds then build is successful.


2.   Extract Elasticsearch tar file
	
	```
	cd /<source_root>/elasticsearch-2.4.1/distribution/tar/target/releases
	tar -xvf elasticsearch-2.4.1-SNAPSHOT.tar.gz
	cd elasticsearch-2.4.1-SNAPSHOT
	```

_**Note:**_  Click [here](https://www.elastic.co/downloads/elasticsearch) to know more about starting the Elasticsearch service.

#### References:
https://www.elastic.co/downloads/elasticsearch