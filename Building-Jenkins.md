<!---PACKAGE:Jenkins--->
<!---DISTRO:SLES 12:2.32--->
<!---DISTRO:SLES 11:2.32--->
<!---DISTRO:RHEL 7.1:2.32--->
<!---DISTRO:RHEL 6.6:2.32--->
<!---DISTRO:Ubuntu 16.x:2.32--->

# Building Jenkins

The instructions provided below specify the steps to build Jenkins version 2.32.1 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

#### Step 1: Install the dependencies

*	RHEL 6.8 and RHEL 7.1/7.2/7.3
     
            sudo yum install -y git 

*	SLES 11-SP4 and SLES 12/12-SP1/12-SP2

            sudo zypper install -y git-core

* 	Ubuntu 16.04/16.10
            
			sudo apt-get install -y git wget
            
* To install Maven
 
	RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4 and SLES 12/12-SP1/12-SP2
          Please refer to the [Maven](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven) recipe.
 
	Ubuntu 16.04/16.10
	  
			sudo apt-get install -y maven
			export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=1024m"	  
		  
* To install Java, use the commands below(Ubuntu 16.04/16.10)

			sudo apt-get install openjdk-8-jdk
 			export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
			export PATH=$JAVA_HOME/bin:$PATH

* To install nodejs and npm

	SLES 11-SP4
		
			sudo zypper install -y awk
 
	RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4 and SLES 12/12-SP1/12-SP2
 
	Download the IBM Node.js SDK for Linux on System z 64-bit binary from [here](https://developer.ibm.com/node/sdk/#v4) and use the command below to install Node.js
		
			chmod +x ibm-4.4.7.0-node-v4.4.7-linux-s390x.bin
			./ibm-4.4.7.0-node-v4.4.7-linux-s390x.bin
	
	Follow screen instruction to install `Node.js` to a folder (for example, `IBM_NODE_HOME`) and set PATH
	
			export PATH=$PATH:<IBM_NODE_HOME>/ibm/node/bin
	
	Ubuntu 16.04/16.10
 	
			sudo apt-get install nodejs npm
			sudo ln -s `which nodejs` /usr/local/bin/node
    
	_**Note:** Ensure that `/usr/local/lib` has sufficient user permissions._

#### Step  2: Build Jenkins
* Get the source code

			cd /<source_root>/
		    git clone https://github.com/jenkinsci/jenkins.git
			cd /<source_root>/jenkins
			git checkout jenkins-2.32.1

* Nodejs requires GCC 4.8 or newer.RHEL 6.8 and SLES 11-SP4 ship older versions of GCC, so it is necessary to build and install a newer GCC.Refer the "Building and Installing GCC" part of the  [GCCGO](https://github.com/linux-on-ibm-z/docs/wiki/Building-gccgo) recipe.
  
* Build the gulp and node modules

			cd /<source_root>/jenkins/war
		    npm install -g gulp gulp-cli (install as a root user on Ubuntu 16.04/16.10)
			npm install --save-dev gulp
			npm install handlebars jenkins-js-modules window-handle bootstrap-detached jenkins-handlebars-rt jenkins-js-test jquery-detached
			npm install jenkins-js-builder
			gulp bundle
	
* Build the source code using mvn command  
   
   Comment out the nodejs plugin download and installation section in `/<source_root>/jenkins/war/pom.xml` file as follows for jenkins war to build succesfully.Nodejs has already been installed in the step above
							
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
	
	**_Note:_** _If the build fails due to timeout, run the command with `-Dfindbugs.timeout=<milliseconds>` parameter.The default value of findbugs plugin is 10 minutes(600,000ms)._
	
* To execute test cases (Optional)

			mvn clean install -pl war -am
	
	_**Note:** User can ignore intermittent test-case failures as it does not affect the functionality (jenkins.security.DefaultConfidentialStoreTest, hudson.LauncherTest and hudson.util.io.TarArchiverTest are the test cases that fail due to wrongly set file path permissions)._
	
	After building jenkins, a ``jenkins.war`` will be created in jenkins/war/target folder.
		   
### References:
https://github.com/jenkinsci/jenkins.git
