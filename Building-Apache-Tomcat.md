<!---PACKAGE:Apache Tomcat--->
<!---DISTRO:SLES 12:8.0.32--->
<!---DISTRO:SLES 11:8.0.32--->
<!---DISTRO:RHEL 7.1:8.0.32--->
<!---DISTRO:RHEL 6.6:8.0.32--->

# Building Apache Tomcat

Apache Tomcat version 8.0.32 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1/6.6 and SLES 12/11.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

#### Step 1: Install the Dependencies

* RHEL6.6:

			sudo yum install -y wget git tar java-1.7.1-ibm-devel.s390x make gcc
						
* RHEL7.1:

			sudo yum install -y wget git tar java-1.8.0-openjdk-devel.s390x make gcc
			
* SLES12:

			sudo zypper install -y wget git-core tar java-1.7.0-openjdk-devel.s390x make gcc

* SLES11:

			sudo zypper install -y wget tar git-core java-1_7_0-ibm-devel-1.7.0_sr9.10-9.1.s390x make gcc

* Install OpenSSL version 1.0.2d using following steps for RHEL7.1/6.6 and SLES12/11

			 cd /<source_root>/
             git clone https://github.com/openssl/openssl.git
             cd openssl
             git checkout OpenSSL_1_0_2d
             ./config --prefix=/usr --openssldir=/usr/local/openssl shared
             make
             sudo make install

* Install Ant using following steps for RHEL7.1/6.6 and SLES12/11:

    Download a binary distribution of Ant from:
		http://ant.apache.org/bindownload.cgi
			
			cd /<source_root>/
			wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.6-bin.tar.gz 
			tar -zxvf apache-ant-1.9.6-bin.tar.gz

#### Step 2: Set Environment Variables
* Set JAVA_HOME and PATH

	* RHEL6.6:

				export JAVA_HOME=/usr/lib/jvm/java
				export PATH=$PATH:$JAVA_HOME/bin

	* RHEL7.1:

				export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
				export PATH=$PATH:$JAVA_HOME/bin

	* SLES12:

				export JAVA_HOME=/usr/lib64/jvm/java-1.7.0-openjdk
				export PATH=$PATH:$JAVA_HOME/bin
	* SLES11:

				export JAVA_HOME=/usr/lib64/jvm/java-1.7.0-ibm-1.7.0
				export PATH=$PATH:$JAVA_HOME/bin
			
* Set ANT_HOME and PATH (For RHEL7.1/6.6 and SLES12/11)

				export ANT_HOME=/<source_root>/apache-ant-1.9.6
				export PATH=$PATH:$ANT_HOME/bin
        
#### Step 3: Build and Install Apache Tomcat
* Get the source
    
			cd /<source_root>/
			git clone https://github.com/apache/tomcat80.git
			cd /<source_root>/tomcat80
			git checkout TOMCAT_8_0_32
		
* Create build properties file

			cd /<source_root>/tomcat80
			cp build.properties.default build.properties
		
* Build Tomcat source

			ant

**Note:** `ant` command will download dependencies, when executed the first time. Ensure that you have write permissions to the folder where the dependencies are saved.

* Run Tests (Optional)

			ant test

* Set CATALINA_HOME and PATH

			export CATALINA_HOME=/<source_root>/tomcat80/output/build
			export PATH=$PATH:$CATALINA_HOME/bin
			
* Run the tomcat server

			cd $CATALINA_HOME/bin
			sh catalina.sh start

#### Step 4: Deploy an application on Apache Tomcat
* To deploy an application on Apache Tomcat following are the two ways:-

1. Go to Apache Tomcat Administration UI (http://localhost) .Click on Manager App and login using Tomcat credentials, you will be redirected to Tomcat Web Application Manager (http://localhost/manager/html)  page. Here you can deploy your application.

	Notes:-
	For setting up the Apache Tomcat Administration UI, following changes have to be added in tomcat-users.xml ($CATALINA_HOME/conf/tomcat-users.xml) file.
	
			<role rolename="manager-gui"/> 
			<user username="tomcat" password="tomcat" roles="manager-gui"/>
	
	Restart Apache Tomcat server after making the above changes.
	To restart, stop the server and start again.
			
			cd $CATALINA_HOME/bin
			sh catalina.sh stop
			sh catalina.sh start
	
	(OR)
			
2. Copy the war file in webapps($CATALINA_HOME/webapps) folder and restart the Apache Tomcat Server.			
			
	
#### References:
* http://tomcat.apache.org/
* http://tomcat.apache.org/download-80.cgi
* https://github.com/apache/tomcat/blob/trunk/BUILDING.txt
 
