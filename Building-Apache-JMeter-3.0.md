<!---PACKAGE:Apache JMeter--->
<!---DISTRO:SLES 12:3.0--->
<!---DISTRO:RHEL 7.1:3.0--->
<!---DISTRO:Ubuntu 16.x:3.0--->

# Building Apache JMeter
Below versions of Apache JMeter are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `2.11`
*    Ubuntu 16.10 has `2.13`

The instructions provided below specify the steps to build [Apache JMeter](http://jmeter.apache.org/) version 3 on Linux on the IBM z Systems for RHEL 7.1/7.2, SLES 12/12-SP1 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	 
_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Section 1: Install the following dependencies

RHEL 7.1/7.2:
```
	sudo yum -y update
	sudo yum install -y git java-1.8.0-openjdk-devel.s390x ant subversion
```

SLES 12/12-SP1:
```
	sudo zypper update -y
	sudo zypper install -y  git openssl libtool autoconf make pcre pcre-devel libxml2-devel liberation-fonts ant subversion
```

_*Note: Check if the packages dbus-1-x11 and libX11-xcb1 are installed using the below commands:*_
```
	rpm -qa | grep dbus-1-x11
	rpm -qa | grep libX11-xcb1
```
   If the above packages are installed, use the below command to uninstall:
```
	sudo zypper remove -y dbus-1-x11 libX11-xcb1
```

Ubuntu 16.04/16.10:
```
	sudo apt-get update
	sudo apt-get install -y subversion openjdk-8-jdk ant openssl
```


##### Set environment variables:

Ubuntu 16.04/16.10:
```
	export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
	export PATH=$PATH:$JAVA_HOME/bin
```

### Section 2: Build and Install Apache JMeter

1. Get the source
      ```
      cd /<source_root>
	  svn co http://svn.apache.org/repos/asf/jmeter/tags/v3_0
      ```

2. Build Apache JMeter
      ```
      cd /<source_root>/v3_0 
      ant download_jars
      ant
      ```

3. Make the below changes in bin/testfiles/Bug52310.xml to make it identical to bin/Bug52310.xml
   ``` 
    sed -i "s/asf-logo.png/asf-logo.svg/" /<source_root>/v3_0/bin/testfiles/Bug52310.xml
    sed -i "s/logo.jpg/logo.svg/" /<source_root>/v3_0/bin/testfiles/Bug52310.xml
    sed -i "/twitter.png/d" /<source_root>/v3_0/bin/testfiles/Bug52310.xml
	  ```


4. Run the functional test-suite
   
   ```
     ant test -Djava.awt.headless=true
   ```

### Reference:

http://jmeter.apache.org/
	