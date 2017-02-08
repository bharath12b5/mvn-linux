<!---PACKAGE:Hibernate--->
<!---DISTRO:SLES 12.x:5.2.5--->
<!---DISTRO:SLES 11.x:5.2.5--->
<!---DISTRO:RHEL 7.x:5.2.5--->
<!---DISTRO:RHEL 6.x:5.2.5--->
<!---DISTRO:Ubuntu 16.x:Distro, 5.2.5--->

### Building Hibernate

The instructions provided below specify the steps to build Hibernate 5.2.5 on the IBM z Systems for the following distributions:

* RHEL (6.8, 7.1, 7.2, 7.3)
* SLES (11 SP4, 12, 12 SP1, 12 SP2)
* Ubuntu (16.04, 16.10)


_**General Notes:**_  
*  _When following the steps below please use a standard permission user unless otherwise specified._

*  _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

### Step 1: Install the Dependencies

   * RHEL 6.8
		
	With IBM SDK:
		
			
	     sudo yum install -y git java-1.8.0-ibm java-1.8.0-ibm-devel
			  
				     	
   * RHEL (7.1, 7.2, 7.3)
		
	With IBM SDK:
		
			
		 sudo yum install -y git java-1.8.0-ibm java-1.8.0-ibm-devel
			
	With Open JDK:
		
			
		 sudo yum install -y git java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x

	
   * SLES (11 SP4, SP 12)
		
	With IBM SDK:
		
		     
         sudo zypper install -y git wget 
			
		
	
    *  Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.  

   			
   * SLES (12 SP1, 12 SP2)
		
	With IBM SDK:
		
			
	     sudo zypper install -y git java-1_8_0-ibm java-1_8_0-ibm-devel
			
	With Open JDK:
		
			
		 
         sudo zypper install -y git java-1_8_0-openjdk java-1_8_0-openjdk-devel      
		
   * Ubuntu (16.04, 16.10)
	
	With IBM SDK:   

	
		sudo apt-get update
		sudo apt-get install -y git wget
			
      *  Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.  

    	
	With Open JDK:
	
		sudo apt-get update
		sudo apt-get install -y git openjdk-8-jdk
			
### Step 2: Set Environment Variables

  ```bash
     export JAVA_HOME=<path to java>
     export PATH=$JAVA_HOME:$PATH 
     export JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF8
     export _JAVA_OPTIONS=-Xmx10g
  ```
  

### Step 3: Build and install Hibernate
  * Get the source

     ```bash
	cd /<source_root>/
	git clone git://github.com/hibernate/hibernate-orm.git
    cd hibernate-orm
    git checkout 5.2.5
    ```
	
  * Change gradle/wrapper/gradle-wrapper.properties to include gradle 3.1 binary (For IBM SDK only)
    ```diff
    @@ -3,4 +3,4 @@ distributionBase=GRADLE_USER_HOME
    distributionPath=wrapper/dists
    zipStoreBase=GRADLE_USER_HOME
    zipStorePath=wrapper/dists
    -distributionUrl=https\://services.gradle.org/distributions/gradle-3.0-milestone-1-bin.zip
    +distributionUrl=https\://services.gradle.org/distributions/gradle-3.1-bin.zip
    ```

  * Build Hibernate  
    ```bash
	 mkdir -p $HOME/.gem/jruby/1.9
     ./gradlew build -x :documentation:renderGettingStartedGuides -x :documentation:renderIntegrationGuide -x :documentation:renderUserGuide -x test
    ```  
	
### Step 4: Run Testcases (Optional)
   
   * With IBM SDK
   ```bash
     ./gradlew test -Dtests.haltonfailure=false --continue
   ```  
	
   * With Open JDK
   ```bash
    ./gradlew test 
   ```
	
_**Note:** With IBM SDK, there are test case failures  which are also observed on Intel x86 VM. These failures can be ignored.
2 tests from CriteriaLockingTest and 1 test from LockNoneWarmingTest fail with IBM SDK._
  
### References  
https://github.com/hibernate/hibernate-orm/tree/5.2.5