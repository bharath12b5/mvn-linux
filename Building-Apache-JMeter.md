# Building Apache JMeter

**Apache JMeter 2.13** has been successfully built and tested on Linux on z Systems. The following instructions can be used for RHEL 7.1/6.6 and SLES 12/11.

### Version
2.13

_**General Notes:**_ 	 
_When following the steps below please use a standard permission user unless otherwise specified._


### Section 1: Install the following Dependencies
*	git (or git-core)
*	openssl
*	libtool
*	autoconf
*	make
*	pcre
*	pcre-devel
*	libxml2-devel
*	Ant
*	java
	

* RHEL6.6:
	```
	yum -y update 
	yum install -y git wget tar java
	```
	
	
* RHEL7.1:
	```
	yum -y update 
	yum install -y git \
				   java \
				   ant
	```

* SLES12:
	```
	zypper -n update
	zypper install -y  git \
					openssl \
					libtool \
					autoconf \
					make \
					pcre \
					pcre-devel \
					libxml2-devel\
					ant

	```

* SLES11:
	```
	zypper -n update
	zypper install -y  git \
					openssl \
					libtool \
					autoconf \
					make \
					pcre \
					pcre-devel \
					libxml2-devel \
					tar \
					java-1_7_0-ibm-devel
	```

* Install Ant on SLES11/RHEL6.6 using following steps:-				   
	```
			wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.3-bin.tar.gz 
			tar zxf apache-ant-1.9.3-bin.tar.gz
	```

	Set environment variables on SLES11:		
	```
			export ANT_HOME=/apache-ant-1.9.3
			export PATH=$PATH:$ANT_HOME/bin
	```

	Set environment variables on RHEL6.6:
	```			
			export ANT_HOME=/apache-ant-1.9.3
			export PATH=$PATH:$ANT_HOME/bin
			export JAVA_HOME=/usr/lib/jvm/java-1.7.1-ibm-1.7.1.3.10.s390x/
			export PATH=$PATH:$JAVA_HOME/bin
	```

### Section 2: Build and install Apache JMeter
1. Get the source

        git clone https://github.com/apache/jmeter.git
        cd /jmeter && git checkout v2_13
		
3. Build Apache JMeter

        cd /jmeter && ant download_jars && ant 
		
4. Run the functional test suite

        ant test -Djava.awt.headless=true

### Reference:

http://jmeter.apache.org/
	