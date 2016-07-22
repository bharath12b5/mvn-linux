<!---PACKAGE:WildFly--->
<!---DISTRO:RHEL 7.1:10--->
<!---DISTRO:Ubuntu 16.x:10--->

### Building WildFly 10.0.0.Final

The instructions provided below specify the steps to build WildFly 10 on Linux on the IBM z Systems for RHEL 7 or Ubuntu 16.04. More information on WildFly is available at https://wildfly.org and the source code can be downloaded from https://github.com/wildfly/wildfly.git

_**General Notes:**_

i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions. This is a temporary writable directory anywhere you'd like to place it._

### Section 1: Install the following dependencies
	
RHEL7:
```
sudo yum install -y git wget tar java-1.8.0-openjdk-devel
```

Ubuntu16.04:
```
sudo apt-get install -y git wget tar maven openjdk-8-jdk 
```


### Section 2: Build and Install WildFly 10
1. Clone the source from Git 

		cd <source_dir>
		git clone https://github.com/wildfly/wildfly.git
		cd wildfly
		git checkout 10.0.0.Final
		cd <source_dir>
        
2. Download apache-maven 3.3.3 (for RHEL7 only)

		wget https://archive.apache.org/dist/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
		tar -xvzf apache-maven-3.3.3-bin.tar.gz
		
3. Set the environment variables 

		export PATH=$PATH:<source_dir>/wildfly/build/target/wildfly-10.0.0.Final/bin
		export MAVEN_OPTS="-Xmx512m -XX:MaxPermSize=1024m"
		export JBOSS_HOME=<source_dir>/wildfly/build/target/wildfly-10.0.0.Final/
		
  RHEL7:
  ```
		export PATH=$PATH:<source_dir>/apache-maven-3.3.3/bin
		export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
		export PATH=$PATH:$JAVA_HOME/bin
  ```
  Ubuntu16.04:
  ```  
		export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x/jre
		export PATH=$PATH:$JAVA_HOME/bin
  ```

4. Build the WildFly 10 source
        
		mvn install	
	
5. Run the server 
	
	Since WildFly server cannot be directly accessed with the default configuration, the local host binding needs to be changed from
	127.0.0.1 to 0.0.0.0 using the commands below
	
	``` 
	Standalone Mode
	sudo sh ./dist/target/wildfly-10.0.0.Final/bin/standalone.sh -b 0.0.0.0 -bmanagement 0.0.0.0
			
	Domain Mode
	sudo sh ./dist/target/wildfly-10.0.0.Final/bin/domain.sh -b 0.0.0.0 -bmanagement 0.0.0.0
	```		
	
After starting WildFly, direct your Web browser to the WildFly Console at: http://<HOST_IP>:8080

### References:
https://wildfly.org