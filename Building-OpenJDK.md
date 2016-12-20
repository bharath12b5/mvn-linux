<!---PACKAGE:OpenJDK--->
<!---DISTRO:SLES 12:9--->
<!---DISTRO:SLES 11:9--->
<!---DISTRO:RHEL 7.1:9--->
<!---DISTRO:RHEL 6.6:9--->
<!---DISTRO:Ubuntu 16.x:9--->

# Building OpenJDK9

Below versions of OpenJDK9 are available in respective distributions at the time of this recipe creation:   
* Ubuntu 16.10 has `OpenJDK9 Zero Variant` 

The instructions provided below specify the steps to build OpenJDK9 on Linux on the IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	
* _When following the steps below please use a super user - root._  
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._  

##Step 1: Building and installing OpenJDK9
####1.1) Install the build dependencies 

  * RHEL 7.1/7.2/7.3
	
  ```
  yum install java-1.8.0-openjdk-devel file-devel unzip zip gcc make gcc-c++ libXtst-devel libXt-devel libXrender-devel libXi-devel cups-devel freetype-devel alsa-lib-devel tar which hg libffi-devel
  ```

  * SLES 12-SP1/12-SP2

  ```
  zypper install java-1_8_0-openjdk-devel hg wget tar make unzip zip gcc gcc-c++ libXtst-devel libXt-devel libXrender-devel libXi-devel cups-devel freetype-devel freetype2-devel alsa-lib-devel libffi48-devel which
  ```

  * Ubuntu 16.04/16.10

  ```
  apt-get update
  apt-get install openjdk-8-jdk make unzip zip  mercurial libx11-dev libxext-dev libxrender-dev libxtst-dev libxt-dev libcups2-dev libfreetype6-dev libxft-dev libasound2-dev  libffi-dev
  ```
  _**Note:** GCC 6 has build issue with OpenJDK9. Make sure you have lower version of GCC while building OpenJDK. This recipe was built and tested with GCC 4.8.5 and 5.4.1._
  
####1.2) Download the source code for OpenJDK9
  ```
  mkdir /<source_root>/
  cd /<source_root>/
  hg clone http://hg.openjdk.java.net/jdk9/hs openjdk9
  cd /<source_root>/openjdk9
  sh get_source.sh
  ```

####1.3) Set the environment variables
  ```
  unset JAVA_HOME
  export LANG=C
  export PATH="/usr/lib/jvm/java-1.8.0-openjdk/bin:${PATH}" (for RHEL 7.1/7.2/7.3)
  export PATH="/usr/lib64/jvm/java-1.8.0-openjdk-1.8.0/bin:${PATH}" (for SLES 12-SP1/12-SP2)
  export PATH="/usr/lib/jvm/java-1.8.0-openjdk-s390x/bin:${PATH}" (for Ubuntu 16.04/16.10)
  ```
    
####1.4) Build OpenJDK9
   * JVM variant - Server (With JIT)

	```
	cd /<source_root>/openjdk9
	bash ./configure --with-jvm-variants=server --disable-warnings-as-errors
	make all
	```
   * JVM variant - Zero (Without JIT) (**NOT** required for Ubuntu 16.10)

	```
	cd /<source_root>/openjdk9
	bash ./configure --with-jvm-variants=zero --disable-warnings-as-errors
	make all
	```

####1.5) Set the `JAVA_HOME` environment variable
  ```
  export JAVA_HOME=/<source_root>/openjdk9/build/linux-s390x-normal-*-release/jdk (* indicates the variant, i.e. server/zero)
  export PATH=$JAVA_HOME/bin:$PATH
  ```

####1.6) Verify java version
  ```
  java -version
  ```

## Step 2: Testing (Optional)
  When the build is completed, you should see the generated binaries and associated files in the `images/jdk` directory in the output directory. In particular, the `build/*/images/jdk/bin` directory should contain executables for the OpenJDK tools and utilities for that configuration. The testing tool `jtreg` will be needed and can be found at: [the jtreg site](http://openjdk.java.net/jtreg/). The provided regression tests in the repositories can be run with the command:
  ```
  cd /<source_root>/openjdk9/test
  make PRODUCT_HOME=/<source_root>/openjdk9/build/linux-s390x-normal-*-release/images/jdk all (* indicates the variant, i.e. server/zero)
  ```

##References:
 http://openjdk.java.net/    
 
