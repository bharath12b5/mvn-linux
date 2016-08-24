<!---PACKAGE:Apache JMeter--->
<!---DISTRO:SLES 11:2.13--->
<!---DISTRO:RHEL 6.6:2.13--->

# Building Apache JMeter

The instructions provided below specify the steps to build Apache JMeter 2.13 on Linux on the IBM z Systems for RHEL6.6 & SLES11:

_**General Notes:**_ 	 
_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Section 1: Install the following dependencies

RHEL6.6:
```
	sudo yum install -y git wget tar java-1.7.1-ibm-devel.s390x subversion
```

SLES11:
```
	sudo zypper refresh
	sudo zypper install -y  git wget openssl libtool autoconf make pcre pcre-devel libxml2-devel tar java-1_7_0-ibm-devel subversion
```


##### Note: Check if the packages dbus-1-x11 and libX11-xcb1 are installed using the below commands:
```
	rpm -qa | grep dbus-1-x11
	rpm -qa | grep libX11-xcb1
```
   If the above packages are installed, use the below command to uninstall:
```
	sudo zypper remove -y dbus-1-x11 libX11-xcb1
```

##### Install Ant on SLES11 and RHEL6.6 only using following steps:
```
    cd /<source_root>
	wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.3-bin.tar.gz 
	tar zxf apache-ant-1.9.3-bin.tar.gz
```


##### Set environment variables:

SLES11:
```
	export ANT_HOME=/<source_root>/apache-ant-1.9.3
	export PATH=$PATH:$ANT_HOME/bin
	export JAVA_HOME=/usr/lib64/jvm/java
```

RHEL6.6:
```
	export ANT_HOME=/<source_root>/apache-ant-1.9.3
	export PATH=$PATH:$ANT_HOME/bin
	export JAVA_HOME=/usr/lib/jvm/java
	export PATH=$PATH:$JAVA_HOME/bin
```

### Section 2: Build and Install Apache JMeter

1. Get the source
      ```
      cd /<source_root>
	  svn co http://svn.apache.org/repos/asf/jmeter/tags/v2_13
      ```

2. Build Apache JMeter
      ```
      cd /<source_root>/v2_13 
      ant download_jars
      ant
      ```

3. Make the below changes in bin/testfiles/Bug52310.xml to make it identical to bin/Bug52310.xml
	  ```
	  sed -i "s/style.css/new-style.css/" /<source_root>/v2_13/bin/testfiles/Bug52310.xml
	  sed -i "s/asf-logo.png/asf-logo.svg/" /<source_root>/v2_13/bin/testfiles/Bug52310.xml
      sed -i "s/logo.jpg/logo.svg/" /<source_root>/v2_13/bin/testfiles/Bug52310.xml
      sed -i "/twitter.png/d" /<source_root>/v2_13/bin/testfiles/Bug52310.xml
	  ```
	  
4. Run the functional test-suite
	  ```
      ant test -Djava.awt.headless=true
      ```

### Reference:

http://jmeter.apache.org/
	