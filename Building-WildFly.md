### Building WildFly 10

_**General Notes:**_ 	 
_When following the steps below please use a standard permission user unless otherwise specified._

The following build instructions have been tested with WildFly 10 on RHEL7 for Linux on z Systems.

### Version
10.0

### Section 1: Install the following dependencies
*	jdk 1.8
*	maven 3.3.3
	
RHEL7:
```
yum install -y git  
yum install -y wget  
yum install -y tar  
yum install -y java-1.8.0-openjdk-devel

```

### Section 2: Build and install WildFly 10
1. Clone the source from Git

        git clone https://github.com/wildfly/wildfly.git
		git checkout -t 10.0.0.CR4
        
2. Download apache-maven 3.3.3

        wget http://www.eu.apache.org/dist/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
        tar -xvzf apache-maven-3.3.3-bin.tar.gz
		
3. Set the environment variables

        export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
        export PATH=$PATH:$JAVA_HOME/bin
        export PATH=$PATH:/wildfly/apache-maven-3.3.3/bin
        export PATH=$PATH:/wildfly/build/target/wildfly-10.0.0.CR1-SNAPSHOT/bin
        export MAVEN_OPTS="-Xmx512m -XX:MaxPermSize=1024m"
        export JBOSS_HOME=/wildfly/build/target/wildfly-10.0.0.CR1-SNAPSHOT
	
4. Build the WildFly 10 source

        mvn install

5. Change local host binding

	Since WildFly server cannot be directly accessed with the default configuration
	in order to get it accessible the local host binding needs to be changed from
	127.0.0.1 to 0.0.0.0 in the following files
	wildfly-10.0.0.CR1-SNAPSHOT/standalone/configuration/standalone.xml and 
	wildfly-10.0.0.CR1-SNAPSHOT/standalone/configuration/domain.xml
	
6. Run the server 

	    Standalone Mode
	    sh <wildfly>/build/src/main/resources/bin/standalone.sh
	    Domain Mode
	    sh <wildfly>/build/src/main/resources/bin/domain.sh
		
### References:
https://wildfly.org