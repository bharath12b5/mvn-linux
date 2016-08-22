<!---PACKAGE:Apache Tomcat--->
<!---DISTRO:SLES 12:8.5.3--->
<!---DISTRO:SLES 11:8.5.3--->
<!---DISTRO:RHEL 7.1:8.5.3--->
<!---DISTRO:RHEL 6.6:8.5.3--->
<!---DISTRO:Ubuntu 16.x:8.5.3--->

# Building Apache Tomcat

Below versions of Apache Tomcat are available in respective distributions at the time of this recipe creation:

*    RHEL   7     has `7.0.54`
*    SLES   12    has `7.0.68`
*    Ubuntu 16.04 has `8.0.32`

The instructions provided below specify the steps to build Apache Tomcat v8.5.3 on Linux on the IBM z Systems for RHEL 6/7, SLES 11/12 and Ubuntu 16.04.

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
			
* Ubuntu 16.04:

			sudo apt-get install -y openjdk-8-jdk tar git make gcc ant
			
						

* Install OpenSSL version 1.0.2h using following steps

			 cd /<source_root>/
             git clone https://github.com/openssl/openssl.git
             cd openssl
             git checkout OpenSSL_1_0_2h
			 
             ./config --prefix=/usr --openssldir=/usr/local/openssl shared
             make
             sudo make install

* Install Ant using following steps (For RHEL7.1/6.6 and SLES12/11 only.)

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
			
	* Ubuntu 16.04:

				export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
				export PATH=$PATH:$JAVA_HOME/bin
					
* Set ANT_HOME and PATH (For RHEL7.1/6.6 and SLES12/11 only.)

				export ANT_HOME=/<source_root>/apache-ant-1.9.6
				export PATH=$PATH:$ANT_HOME/bin
        
#### Step 3: Build and Install Apache Tomcat
* Get the source
    
			cd /<source_root>/
			git clone https://github.com/apache/tomcat85.git
			cd /<source_root>/tomcat85
			git checkout TOMCAT_8_5_3
		
* Create build properties file

			cd /<source_root>/tomcat85
			cp build.properties.default build.properties
		
* Build Tomcat source

			ant

**Note:** `ant` command will download dependencies, when executed the first time. Ensure that you have write permissions to the folder where the dependencies are saved.

* Run Tests (Optional)

			ant test

* Set CATALINA_HOME and PATH

			export CATALINA_HOME=/<source_root>/tomcat85/output/build
			export PATH=$PATH:$CATALINA_HOME/bin
			
* Run the tomcat server

			cd $CATALINA_HOME/bin
			sh catalina.sh start		
	
#### References:
* http://tomcat.apache.org/
* http://tomcat.apache.org/download-85.cgi
* https://github.com/apache/tomcat/blob/trunk/BUILDING.txt
* https://tomcat.apache.org/tomcat-8.5-doc/manager-howto.html
 
