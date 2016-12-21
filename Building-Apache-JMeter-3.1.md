<!---PACKAGE:Apache JMeter--->
<!---DISTRO:SLES 12:3.1--->
<!---DISTRO:RHEL 7.1:3.1--->
<!---DISTRO:Ubuntu 16.x:3.1--->

# Building Apache JMeter
Below versions of Apache JMeter are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `2.11`
*    Ubuntu 16.10 has `2.13`

The instructions provided below specify the steps to build [Apache JMeter](http://jmeter.apache.org/) version 3 on Linux on the IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	 
*_When following the steps below please use a standard permission user unless otherwise specified._

*_A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Step 1: Building and Installing ApacheJMeter

####1.1) Install dependencies

RHEL 7.1/7.2/7.3:

```
	sudo yum -y update
	sudo yum install -y git java-1.8.0-openjdk-devel.s390x ant subversion
```

SLES 12/12-SP1/12-SP2:

```
	sudo zypper update -y
	sudo zypper install -y  git openssl libtool autoconf make pcre pcre-devel libxml2-devel liberation-fonts ant subversion
```

Ubuntu 16.04/16.10:

```
	sudo apt-get update
	sudo apt-get install -y subversion openjdk-8-jdk ant openssl
```

####1.2) Set environment variables:

Ubuntu 16.04/16.10:

```
	export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
	export PATH=$PATH:$JAVA_HOME/bin
```

## Step 2: Build and Install Apache JMeter

####2.1)Get the source

```
	cd /<source_root>
	svn co http://svn.apache.org/repos/asf/jmeter/tags/v3_1
```

####2.2) Build Apache JMeter

```
	cd /<source_root>/v3_1
	ant download_jars
	ant
```

####2.3)Run the functional test-suite

```
	ant test -Djava.awt.headless=true
```

## References:

http://jmeter.apache.org/
	