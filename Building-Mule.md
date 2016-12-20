<!---PACKAGE:Mule--->
<!---DISTRO:SLES 12:3.8.x--->
<!---DISTRO:RHEL 7.1:3.8.x--->
<!---DISTRO:Ubuntu 16.x:3.8.x--->

# Building Mule ESB Standalone

The instructions provided below specify the steps to build Mule ESB Standalone version 3.8.2-s390x on IBM z Systems for RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	 
* When following the steps below, please use a standard permission user unless otherwise specified.

* A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it.


1. Install dependencies
	
	RHEL 6.8
	
	* Using IBM-JDK         	
		
		```
		sudo yum install -y wget tar unzip git make gcc java-1.7.1-ibm java-1.7.1-ibm-devel 
		```	
		
	RHEL 7.1/7.2/7.3           
     			
	* Using IBM-JDK	      
		
		```
		sudo yum install -y wget tar unzip git make gcc java-1.8.0-ibm.s390x java-1.8.0-ibm-devel.s390x maven
		```	
	* Using Open-JDK      	
		
		```
		sudo yum install -y wget tar unzip git make gcc java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x maven  
		```
	
	SLES 11-SP4      

	* Using IBM-JDK       	
		
		```
		sudo zypper install -y wget tar unzip git make gcc java-1_7_0-ibm-devel
		```		
	
	SLES 12       

	* Using IBM-JDK      	
		
		```
		sudo zypper install -y wget tar unzip git make gcc java-1_7_1-ibm-devel
		```	
	* Using Open-JDK       
	
		```
		sudo zypper install -y wget tar unzip git make gcc java-1_7_0-openjdk-devel
		```
	
    SLES 12-SP1/12-SP2        
				
	* Using IBM-JDK       	
		
		```
		sudo zypper install -y wget tar unzip git make gcc java-1.8.0-ibm.s390x java-1.8.0-ibm-devel.s390x
		```
	* Using Open-JDK        
	
		```
		sudo zypper install -y wget tar unzip git make gcc java-1_8_0-openjdk java-1_8_0-openjdk-devel
		```
		
	Ubuntu 16.04/16.10
    
	* Using IBM-JDK	
		
		```        
		sudo apt-get install -y wget tar unzip git make gcc maven
		```
		
		Install IBM Java 8
	
		Download IBM Java 8 SDK binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per the given link.
	  
	* Using Open-JDK        	
		
		```
        sudo apt-get install -y wget tar unzip git openjdk-8-jdk openjdk-8-jre make gcc maven
		```	
	
2.  Install Maven (RHEL 6.8, SLES 11-SP4/12/12-SP1/12-SP2)
	
		cd /<source_root>/
		wget https://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
		tar -xvzf apache-maven-3.3.9-bin.tar.gz
		export PATH=/<source_root>/apache-maven-3.3.9/bin:$PATH
	
3.  Set the environment variables

	RHEL 6.8 (Using IBM-JDK)
	
		export JAVA_HOME=/usr/lib/jvm/java-1.7.1-ibm.s390x
		export PATH=$JAVA_HOME/bin:$PATH
		
	RHEL 7.1/7.2/7.3 (Using IBM-JDK)		
	
		export JAVA_HOME=/usr/lib/jvm/java-1.8.0-ibm
		export PATH=$JAVA_HOME/bin:$PATH  
	
	RHEL 7.1/7.2/7.3 (Using Open-JDK)

		export JAVA_HOME=/usr/lib/jvm/java
		export PATH=$JAVA_HOME/bin:$PATH
	
	SLES 11-SP4 (Using IBM-JDK)
	
		export JAVA_HOME=/usr/lib64/jvm/java-1.7.0-ibm
		export PATH=$JAVA_HOME/bin:$PATH
		
	SLES 12 (Using IBM-JDK)
	
		export JAVA_HOME=/usr/lib64/jvm/java-1.7.1-ibm
		export PATH=$JAVA_HOME/bin:$PATH
	
	SLES 12-SP1/12-SP2 (Using IBM-JDK) 

		export JAVA_HOME=/usr/lib64/jvm/java-1.8.0-ibm
		export PATH=$JAVA_HOME/bin:$PATH
		
	SLES 12/12-SP1/12-SP2 (Using Open-JDK)
		
		export JAVA_HOME=/usr/lib64/jvm/java    
	    export PATH=$JAVA_HOME/bin:$PATH
	
	Ubuntu 16.04/16.10 (Using IBM-JDK)
		
		export JAVA_HOME=/<user_install_dir>/ibm/java
 
      _**Note:** Where `/<user_install_dir>/` is the location where IBM SDK is installed. Ideally the location is `/opt`._
	
	Ubuntu 16.04/16.10 (Using Open-JDK)      		
	
		export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-s390x
				
4.  Checkout the Mule ESB Standalone mule-3.8.2 for s390x source

		cd /<source_root>/
		git clone https://github.com/linux-on-ibm-z/mule
		cd mule
		git checkout mule-3.8.2-s390x
		
5.  Add the location of tls-default.conf to PATH  

		export PATH=$PATH:/<source_root>/mule/distributions/standalone/src/main/resources/conf

6.  Generate SSL certificate   

        cd /<source_root>/mule/transports/ssl
		keytool -selfcert -alias Test -genkey -keystore myStore.keystore -keyalg RSA -validity 1
    
	_**Note:**The above command will prompt for keystore password of your choice and details such as name of your organizational unit etc_

7.  Build the code
        
		cd /<source_root>/mule/
		
	To build individual jar files and execute test cases run the following command    
		
		mvn clean install -Pdistributions -fn     
				
	
	To build without running test cases execute the following    
		
		mvn clean install -Pdistributions -DskipTests -fn
				
	
	 _**Note:** On successfull build `mule-standalone-3.8.2.tar.gz` file will be created in the path `<source_root>/mule/distributions/standalone/target/mule-standalone-3.8.2.tar.gz`_

8.  Extract the mule-standalone-3.8.2 tar file and add to the path

		tar xzvf /<source_root>/mule/distributions/standalone/target/mule-standalone-3.8.2.tar.gz -C /<source_root>/
		export PATH=$PATH:/<source_root>/mule-standalone-3.8.2/bin
		
9.  Mule requires Java Service Wrapper.
    The Wrapper is available at http://wrapper.tanukisoftware.com/doc/english/download.jsp in standard and professional editions which require a [licence](https://wrapper.tanukisoftware.com/doc/english/licenseOverview.html)
    
    Copy the wrapper binary to mule
      
	    cd /<source_root>/
		wget http://wrapper.tanukisoftware.com/download/3.5.30/wrapper-linux-390-64-3.5.30-*.tar.gz  
		tar -zxvf wrapper-linux-390-64-3.5.30-*.tar.gz 
	    cp /<source_root>/wrapper-linux-390-64-3.5.30-*/bin/wrapper /<source_root>/mule-standalone-3.8.2/lib/boot/exec
	
	_**Note:** The `*` in the above links refer to Standard/Professional editions of the Java Service Wrapper_
	
10. Start mule server 
	
		mule

    _**Note:** The server starts with the below output_
	```
	INFO  2016-07-01 05:47:22,192 [WrapperListener_start_runner] org.mule.module.launcher.DeploymentDirectoryWatcher:
	++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
	+ Mule is up and kicking (every 5000ms)                    +
	++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
    ```
To know about deploying mule applications refer https://docs.mulesoft.com/mule-fundamentals/v/3.8/deploying-mule-applications