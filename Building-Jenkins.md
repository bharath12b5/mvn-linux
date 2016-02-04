# Building Jenkins

Jenkins version 1.625.3 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 6.6, RHEL 7.1, SLES11 and SLES 12.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##### Step 1: Install the dependencies
Following are the build dependencies for Jenkins:

* git (RHEL6/7) or git-core (SLES11/12)
* Maven 

**Dependencies Installation Notes:**   
* Git can be installed on RHEL6/7 using the command below.
     
            sudo yum install -y git

* Git can be installed on SLES11/12 using the command below.

            sudo zypper install -y git-core
            
* To install Maven, please refer to the [Maven](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven) recipe.

##### Step 2: Build Jenkins
* Get the source (clone branch 1.625.3)  

			cd /<source_root>/
		    git clone https://github.com/jenkinsci/jenkins.git
			cd /<source_root>/jenkins
			git checkout jenkins-1.625.3
			
* Build the source code using mvn command
	
			cd /<source_root>/jenkins
			mvn clean install -pl war -am -DskipTests 
			
* To execute test cases [optional]

			mvn clean install -pl war -am
	
	Note: User can ignore intermittent test-case failures as it does not affect the functionality (jenkins.security.DefaultConfidentialStoreTest, hudson.LauncherTest and hudson.util.io.TarArchiverTest are the test cases that fail due to wrongly set file path permissions)
	
	After building jenkins, a ``jenkins.war`` will be created in jenkins/war/target folder.
   	
##### Step 3: Deploy Jenkins 

* Refer to the Deploy section of [Apache Tomcat](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-Tomcat) recipe for deploying jenkins.war on Apache Tomcat.

    
### References:
https://github.com/jenkinsci/jenkins.git
