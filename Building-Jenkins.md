<!---PACKAGE:Jenkins--->
<!---DISTRO:SLES 12:2.0--->
<!---DISTRO:SLES 11:2.0--->
<!---DISTRO:RHEL 7.1:2.0--->
<!---DISTRO:RHEL 6.6:2.0--->
<!---DISTRO:Ubuntu 16.x:2.0--->

# Building Jenkins

Jenkins version 2.0 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 6.6, RHEL 7.1, SLES 11, SLES 12 and Ubuntu 16.04.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##### Step 1: Install the dependencies
Following are the build dependencies for Jenkins:

* git (RHEL6/7 and Ubuntu 16.04) or git-core (SLES11/12)
* Maven 

**Dependencies Installation Notes:**   
* Git can be installed on RHEL6/7 using the command below.
     
            sudo yum install -y git

* Git can be installed on SLES11/12 using the command below.

            sudo zypper install -y git-core

* Git can be installed on Ubuntu 16.04 using the command below
 
            sudo apt-get install -y git
            
* To install Maven
 
 For RHEL6/7 and SLES11/12
          Please refer to the [Maven](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven) recipe.
 
 For Ubuntu 16.04
	  
         sudo apt-get install -y maven
		 export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=1024m"	  
		  
* To install Java, use the command below,(Only for Ubuntu 16.04)

            sudo apt-get install openjdk-8-jdk
 			export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
			export PATH=$JAVA_HOME/bin:$PATH

##### Step 2: Build Jenkins
* Get the source (clone branch 2.0)  

			cd /<source_root>/
		    git clone https://github.com/jenkinsci/jenkins.git
			cd /<source_root>/jenkins
			git checkout jenkins-2.0
			
* Build the source code using mvn command
	
			
			
   Comment out the nodejs plugin download and installation section in `/<source_root>/jenkins/war/pom.xml` file  as follows for jenkins war to build succesfully.
			
			
				
			<!--
			<profile>
			<id>node-classifier-linux</id>
			..................................
			..................................
			-->
			</profiles>
			</project>
   
   
   Build the source
	
			cd /<source_root>/jenkins
			mvn clean install -pl war -am -DskipTests 
			
*  To install nodejs plugin manually [optional]
	
	For RHEL7/RHEL6
	
			sudo yum install make gcc-c++
			
	For SLES12/SLES11
	
			sudo zypper install make gcc-c++
	
	For RHEL7/RHEL6/SLES12/SLES11
	
			cd /<source_root>/ 
			git clone https://github.com/andrewlow/node.git
			cd /<source_root>/node 
			./configure && make && sudo make install
	
	For Ubuntu 16.04
	
			sudo apt-get install nodejs npm
			
* To execute test cases [optional]

			mvn clean install -pl war -am
	
	Note: User can ignore intermittent test-case failures as it does not affect the functionality (jenkins.security.DefaultConfidentialStoreTest, hudson.LauncherTest and hudson.util.io.TarArchiverTest are the test cases that fail due to wrongly set file path permissions)
	
	After building jenkins, a ``jenkins.war`` will be created in jenkins/war/target folder.
   	
##### Step 3: Deploy Jenkins 

* Refer to the Deploy section of [Apache Tomcat](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-Tomcat) recipe for deploying jenkins.war on Apache Tomcat.

    
### References:
https://github.com/jenkinsci/jenkins.git
