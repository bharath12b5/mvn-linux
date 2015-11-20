### Building Apache Tomcat

Apache Tomcat version 8.0.28 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1/6.6 and SLES 12/11.

_**General Notes:**_ 	 
_When following the steps below please use a standard permission user unless otherwise specified._

### Step 1: Install the Dependencies
Following are the build dependencies for Apache Tomcat:

* wget
* tar
* java-1.7.0 (or higher version)
* ant

**Dependency Installation Notes:**   
* RHEL6.6:

			yum install -y wget \
			tar \
			java-1.7.1-ibm-devel.s390x
			
			
* RHEL7.1:

			yum install -y wget \
			tar \
			java-1.8.0-openjdk-devel.s390x


* SLES12:

			zypper install -y wget \
			tar \
			java-1.7.0-openjdk-devel.s390x

* SLES11:

		zypper install -y wget tar java-1_7_0-ibm-devel

* Install Ant using following steps on RHEL7.1/6.6 and SLES12/11:

    Download a binary distribution of Ant from:
		http://ant.apache.org/bindownload.cgi

			wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.3-bin.tar.gz 
			tar zxf apache-ant-1.9.3-bin.tar.gz

### Step 2: Set Environment Variables
* Set JAVA_HOME and PATH

	* RHEL6.6:

				export JAVA_HOME=/usr/lib/jvm/java-1.7.1-ibm-1.7.1.3.10.s390x/
				export PATH=$PATH:$JAVA_HOME/bin

	* RHEL7.1:

				export JAVA_HOME=/usr/lib/jvm/java-1.8.0
				export PATH=$PATH:$JAVA_HOME/bin

	* SLES12:

				export JAVA_HOME=/usr/lib64/jvm/java-1.7.0
				export PATH=$PATH:$JAVA_HOME/bin
	* SLES11:

				export JAVA_HOME=/usr/lib64/jvm/java-1.7.0-ibm-1.7.0
				export PATH=$PATH:$JAVA_HOME/bin
			
* Set ANT_HOME and PATH (For RHEL7.1/6.6 and SLES12/11)

			export ANT_HOME=/apache-ant-1.9.3
			export PATH=$PATH:$ANT_HOME/bin
        
### Step 3: Build and install Apache Tomcat
* Get the source
    
			wget http://apache.mesi.com.ar/tomcat/tomcat-8/v8.0.28/src/apache-tomcat-8.0.28-src.tar.gz
			tar zxf apache-tomcat-8.0.28-src.tar.gz
		
* Create build properties file and log file

			cd /apache-tomcat-8.0.28-src
			cp build.properties.default build.properties
		
* Build Tomcat source

			ant

* Set CATALINA_HOME and PATH

			export CATALINA_HOME=/apache-tomcat-8.0.28-src/output/build
			export PATH=$PATH:$CATALINA_HOME/bin
			
* Run the tomcat server

			cd $CATALINA_HOME/bin
			sh catalina.sh start

### Step 4: Deploy an application on Apache Tomcat
* To deploy an application on Apache Tomcat following are the two ways:-

1. Go to Apache Tomcat Administration UI (http://localhost) . Click on Manager App and login using Tomcat credentials, you will be redirected to Tomcat Web Application Manager (http://localhost/manager/html)  page. Here you can deploy your application.

	Notes:-
	For setting up the Apache Tomcat Administration UI, following changes have to be added in tomcat-users.xml ($CATALINA_HOME/conf/tomcat-users.xml) file.
	
			<role rolename="manager-gui"/> 
			<user username="tomcat" password="tomcat" roles="manager-gui"/>
	
	Restart Apache Tomcat server after making the above changes.
	
	(OR)
			
2. Copy the war file in webapps($CATALINA_HOME/webapps) folder and restart the Apache Tomcat Server.			
			
	
### References:
http://tomcat.apache.org/
http://tomcat.apache.org/download-80.cgi
https://github.com/apache/tomcat/blob/trunk/BUILDING.txt
 