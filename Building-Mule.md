<!---PACKAGE:Mule--->
<!---DISTRO:SLES 12:3.8.x--->
<!---DISTRO:RHEL 7.1:3.8.x--->
<!---DISTRO:Ubuntu 16.x:3.8.x--->

## Building Mule ESB Standalone on RHEL 7, SLES 12-sp1 and Ubuntu 16.04

The instructions provided below specify the steps to build Mule ESB Standalone version 3.8.0-s390x on Linux on the IBM z Systems for RHEL 7, SLES 12-sp1 and Ubuntu 16.04

####General Notes:

i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it. 


1. Install dependencies

	On RHEL 7:
     			
	* Using Open-JDK:
	
		```
		sudo yum install -y wget tar unzip git make gcc java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x 
		```
	* Using IBM-JDK:
	
		```
		sudo yum install -y wget tar unzip git make gcc java-1.8.0-ibm.s390x java-1.8.0-ibm-devel.s390x
		```	
		
    On SLES 12-sp1:
				
	* Using Open-JDK:
		
		```
		sudo zypper install -y wget tar unzip git make gcc java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x 
		```
	* Using IBM-JDK:
	
		```
		sudo zypper install -y wget tar unzip git make gcc java-1.8.0-ibm.s390x java-1.8.0-ibm-devel.s390x
		```
		
	On Ubuntu 16.04:
    
        sudo apt-get install -y  wget tar unzip git openjdk-8-jdk openjdk-8-jre make gcc maven

2.  Install Maven (Only for RHEL7 and SLES12-sp1)
	
		cd /<source_root/
		wget https://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
		tar -xvzf apache-maven-3.3.9-bin.tar.gz
		export PATH=/<source_root>/apache-maven-3.3.9/bin:$PATH
	
3.  Set the environment variables

	On RHEL 7:   		
	
		export JAVA_HOME=/usr/lib/jvm/java    
	   
	
	On SLES 12-sp1:   		
		
	
		export JAVA_HOME=/usr/lib64/jvm/java    
	  
	
	On Ubuntu 16.04:       		
	
		export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-s390x
	
4.  Checkout the Mule ESB Standalone mule-3.8.0 for s390x source

		cd /<source_root>/
		git clone https://github.com/linux-on-ibm-z/mule
		cd  mule
		git checkout mule-3.8.0-s390x
		
5.  Add the location of tls-default.conf to PATH  

		export PATH=$PATH:/<source_root>/mule/distributions/standalone/src/main/resources/conf

6.  Generate SSL certificate   

        cd /<source_root>/mule/transports/ssl
		keytool -selfcert -alias Test -genkey -keystore myStore.keystore -keyalg RSA -validity 1
    
	_**Note:**The above command will prompt for keystore password of your choice and details such as name of your organizational unit etc_

7.  Build the code
        
		cd /<source_root>/mule/
		
	To build individual jar files and execute test cases run the following command    
		
		mvn clean install -DskipDistributions=false -fn     
				
	
	To build without running test cases execute the following    
		
		mvn clean install -DskipDistributions=false -DskipTests -fn
				
	
	 _**Note:** On successfull build `mule-standalone-3.8.0.tar.gz` file will be created in the path `<source_root>/mule/distributions/standalone/target/mule-standalone-3.8.0.tar.gz`_

8.  Extract the mule-standalone-3.8.0 tar file and add to the path

		tar xzvf /<source_root>/mule/distributions/standalone/target/mule-standalone-3.8.0.tar.gz -C /<source_root>/
		export PATH=$PATH:/<source_root>/mule-standalone-3.8.0/bin
		
9.  Mule requires Java Service Wrapper.
    The Wrapper is available at http://wrapper.tanukisoftware.com/doc/english/download.jsp in standard and professional editions which require a [licence](https://wrapper.tanukisoftware.com/doc/english/licenseOverview.html)
    
    Copy the wrapper binary to mule
      
	    cd /<source_root>/
		wget http://wrapper.tanukisoftware.com/download/3.5.30/wrapper-linux-390-64-3.5.30-*.tar.gz  
	    cp /<source_root>/wrapper-linux-390-64-3.5.30-*/bin/wrapper /<source_root>/mule-standalone-3.8.0/lib/boot/exec
	
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