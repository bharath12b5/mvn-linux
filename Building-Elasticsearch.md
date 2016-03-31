<!---PACKAGE:Elasticsearch--->
<!---DISTRO:SLES 12:2.2.1--->
<!---DISTRO:RHEL 7.1:2.2.1--->

# Building Elasticsearch

The instructions provided below specify the steps to build [Elasticsearch](https://www.elastic.co/products/elasticsearch) v2.2.1 on Linux on the IBM z Systems for RHEL 7 and SLES 12.

##### General Notes:
      
i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it.

#### Install Build Dependencies

1. RHEL7
    
    	sudo yum install java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x tar wget unzip
        
2. SLES12
  
    	sudo zypper install java-1_7_0-openjdk java-1_7_0-openjdk-devel tar wget unzip

#### Download Maven and add it to the path

1. Download Maven
	```
    cd /<source_root>/
    wget http://archive.apache.org/dist/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
    tar -zxvf apache-maven-3.3.3-bin.tar.gz
    ```
    
2. Set the Environment variables

	```
    MAVEN="/<source_root>/apache-maven-3.3.3/bin"
    export PATH=$PATH:$MAVEN
    export MAVEN_OPTS="-Xms1024m -Xmx1024m -XX:MaxPermSize=1024m"
    ```
       


3. Increase the maximum number of files that are allowed to be opened on the system. 
   "ulimit -n" should be greater than 32k or 64k. 32k and 64k are the suggested sizes.

	In order to increase this limit, follow the given steps.
 
	As root, edit the file /etc/security/limits.conf, add the given line.
	```
    *       -       nofile      65535
    ```
    
   Also as root, run the command "ulimit -n 65535". 
	Logout and Login. Run the command ulimit -n, it should return 65535.


#### Obtain required version of Elasticsearch

1. Download Elasticsearch

	```
    cd /<source_root>/
    wget https://github.com/elastic/elasticsearch/archive/v2.2.1.zip
    unzip v2.2.1.zip
    cd elasticsearch-2.2.1
    ```
    
#### Build the package

To build the package without the tests: 

		mvn clean package -DskipTests

		
To build with local tests
		
        mvn clean package -Des.node.mode=local
	

To build with the network tests
		
        mvn clean package -Des.node.mode=network

		

The default build (mvn clean package) without any parameters is -Des.node.mode=local 
Elasticseach uses randomized testing, and hence many tests might fail unpredictably, this can be ignored. If the -DskipTests succeeds the build is successful
