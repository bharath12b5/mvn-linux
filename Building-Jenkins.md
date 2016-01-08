### Building Jenkins

Jenkins version 1.625 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 6.6 ,RHEL 7.1 , SLES11 and SLES 12.

_**General Notes:**_ 	 
_When following the steps below please use a standard permission user unless otherwise specified._

### Step 1: Install the dependencies
Following are the build dependencies for Jenkins:

* git (RHEL6/7) or git-core (SLES11/12)
* Maven 3.2.5 (RHEL6/7) or Maven 3.3.3 (SLES11/12)

**Dependencies Installation Notes:**   
* Git can be installed on RHEL6/7 using the below command.
     
            yum install git

* Git can be installed on SLES11/12 using the below command.

            zypper install git-core
            
* To install Maven 3.2.5 (RHEL6/7), please refer to the [Maven](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven) recipe.

* To install Maven 3.3.3 (SLES11/12), please refer to the [Maven](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven) recipe. 

### Step 2: Build Jenkins
* Get the source (clone branch 1.625)  
  
		    git clone -b stable-1.625 https://github.com/jenkinsci/jenkins.git
			
* Build the source code using mvn command
	
			cd jenkins
			mvn clean install -pl war -am -DskipTests 
			
* To execute test cases [optional]

			mvn clean install -pl war -am
	
	Note: User can ignore intermittent test-case failures as it does not effect the functionality (jenkins.security.DefaultConfidentialStoreTest, hudson.LauncherTest and hudson.util.io.TarArchiverTest are the test cases that fail due to wrongly set file path permissions)
	
	After building jenkins, a jenkins.war will be created in jenkins/war/target folder.
   	
### Step 3: Deploy Jenkins 

* Refer to the Deploy section of [Apache Tomcat](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-Tomcat) recipe for deploying jenkins.war on Apache Tomcat.

    
### References:
https://github.com/jenkinsci/jenkins.git
