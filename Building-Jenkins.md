<!---PACKAGE:Jenkins--->
<!---DISTRO:SLES 12:2.7--->
<!---DISTRO:SLES 11:2.7--->
<!---DISTRO:RHEL 7.1:2.7--->
<!---DISTRO:RHEL 6.6:2.7--->
<!---DISTRO:Ubuntu 16.x:2.7--->

# Building Jenkins

The instructions provided below specify the steps to build Jenkins version 2.7.1 on Linux on the IBM z Systems for RHEL6.6 ,RHEL7.1, SLES11, SLES 12 and Ubuntu 16.04:

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

#### Step 1: Install the dependencies
Following are the build dependencies for Jenkins:

* git (RHEL6/7 and Ubuntu 16.04) or git-core (SLES11/12)
* Maven 

**Dependencies Installation Notes:**   
* Git can be installed on RHEL6/7 using the command below.
     
            sudo yum install -y git

* Git can be installed on SLES11/12 using the command below.

            sudo zypper install -y git-core

* Git can be installed on Ubuntu 16.04 using the command below
 
            sudo apt-get install -y git wget
            
* To install Maven
 
 For RHEL6/7 and SLES11/12
          Please refer to the [Maven](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven) recipe.
 
 For Ubuntu 16.04
	  
         sudo apt-get install -y maven
		 export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=1024m"	  
		  
* To install Java, use the command below(Only for Ubuntu 16.04)

            sudo apt-get install openjdk-8-jdk
 			export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
			export PATH=$JAVA_HOME/bin:$PATH

* To install nodejs and npm
 
 For RHEL6/7 and SLES11/12
 
	Download the IBM Node.js SDK for Linux on System z 64-bit binary from [here](https://developer.ibm.com/node/sdk/#v4) and use the command below to install Node.js
		
			chmod +x ibm-4.4.7.0-node-v4.4.7-linux-s390x.bin
			./ibm-4.4.7.0-node-v4.4.7-linux-s390x.bin
	
	Follow screen instruction to install `Node.js` to a folder (for example, `IBM_NODE_HOME`) and set PATH
	
			export PATH=$PATH:<IBM_NODE_HOME>/ibm/node/bin
	
	**_Note:_** _GCC 6 and above is required to install Node.js on RHEL6 and SLES11. Please refer the "Building and Installing GCC" part of the  [GCCGO](https://github.com/linux-on-ibm-z/docs/wiki/Building-gccgo) recipe._
 
 For Ubuntu 16.04
 	
			sudo apt-get install nodejs npm
			sudo ln -s `which nodejs` /usr/local/bin/node

#### Step  2: Build Jenkins
* Get the source (clone branch 2.7.1)  

			cd /<source_root>/
		    git clone https://github.com/jenkinsci/jenkins.git
			cd /<source_root>/jenkins
			git checkout jenkins-2.7.1
	
* Build the gulp and node modules

			cd /<source_root>/jenkins/war
		    npm install -g gulp gulp-cli (install as a root user on Ubuntu 16.04)
			npm install --save-dev gulp
			npm install handlebars jenkins-js-modules window-handle bootstrap-detached jenkins-handlebars-rt jenkins-js-test jquery-detached
			npm install jenkins-js-builder
			gulp bundle

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
	
* To execute test cases [optional]

			mvn clean install -pl war -am
	
	_**Note:** User can ignore intermittent test-case failures as it does not affect the functionality (jenkins.security.DefaultConfidentialStoreTest, hudson.LauncherTest and hudson.util.io.TarArchiverTest are the test cases that fail due to wrongly set file path permissions)_
	
	After building jenkins, a ``jenkins.war`` will be created in jenkins/war/target folder.
		   
### References:
https://github.com/jenkinsci/jenkins.git
