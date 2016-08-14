<!---PACKAGE:Apache Maven--->
<!---DISTRO:SLES 12:3.x--->
<!---DISTRO:SLES 11:3.x--->
<!---DISTRO:RHEL 7.1:3.x--->
<!---DISTRO:RHEL 6.6:3.x--->
<!---DISTRO:Ubuntu 16.x:3.x--->

# Building Maven

Below versions of Maven are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `3.3.9`

The instructions provided below specify the steps to build Maven version 3.3.9 on Linux on the IBM z Systems for RHEL 6.6/7.1, SLES 11/12.

_**General Notes:**_

i) When following the steps below please use a standard permission user unless otherwise specified.
	 
ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it

### Obtain pre-built dependencies

1. Use the following commands to obtain dependencies

    For RHEL 7.1

  ```shell
  sudo yum install ant java-1.7.1-ibm-devel tar wget
  ```
	
    For RHEL 6.6

  ```shell
  sudo yum install java-1.7.1-ibm-devel tar wget
  ```
	
    For SLES 12

  ```shell
  sudo zypper install ant java-1_7_1-ibm-devel tar wget
  ```
	
    For SLES 11

  ```shell
  sudo zypper install java-1_7_0-ibm-devel tar wget
  ```

  Note: Ant is not listed for SLES 11 or RHEL 6.6 as they provide an older version of Ant than is required to build Maven. Hence for SLES 11 and RHEL 6.6, Ant is installed in a subsequent step below.
  
1. Create a temporary installation directory 

  ```shell
  mkdir /<source_root>
  cd /<source_root>/
  export WORK_DIR=`pwd`
  ```

1. For **SLES 11 and RHEL 6.6** install appropriate version of Ant 

  ```shell
  cd $WORK_DIR
  wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.4-bin.tar.gz
  tar -zxvf apache-ant-1.9.4-bin.tar.gz
  export PATH=$WORK_DIR/apache-ant-1.9.4/bin:$PATH
  ```
	
### Product Build - Maven

1. Set environment variables and working directory

  For **RHEL 7.1 and RHEL 6.6**
  
  ```shell
  export JAVA_HOME=/usr/lib/jvm/java
  ```
  
  For **SLES 12 and SLES 11**
  
  ```shell
  export JAVA_HOME=/usr/lib64/jvm/java
  ```
  
  For **RHEL 7.1, RHEL 6.6, SLES 12 and SLES 11**
  
  ```shell
  export M2_HOME=$WORK_DIR/maven
  export PATH=$JAVA_HOME/bin:$PATH:$M2_HOME/bin
  ```
	
1. Download the required Maven source code

  ```shell
  cd $WORK_DIR
  wget http://apache.cs.utah.edu/maven/maven-3/3.3.9/source/apache-maven-3.3.9-src.tar.gz 
  tar -zxvf apache-maven-3.3.9-src.tar.gz
  ```
	
1. Build Maven (using Ant)

    ```shell
    cd $WORK_DIR/apache-maven-3.3.9
    ant
    ```
  On completion of a successful build, a "BUILD SUCCESSFUL" message is output by Ant.
  
  Tests are run as part of the build and should all pass. Test reports can be found under:    
    `$WORK_DIR/apache-maven-3.3.9/maven-compat/target/surefire-reports`


1. Verify the build

  The following commands should display the version of Maven that has been built and output usage information.

  ```shell
  mvn --version
  mvn --help
  ```

1. Install Maven

  The above build creates a Maven installation at $M2_HOME i.e. `$WORK_DIR/maven`. This may then be moved to an alternative location as desired.
  
  If the installation is moved then M2_HOME and PATH variables will need to be updated accordingly.

  The installation may be verified by running the same commands that were used to verify the build (in the previous step).
	

1. _[Optional]_ Clean up

  Once you have a copy of the Maven installation outside $WORK_DIR then $WORK_DIR may be deleted.
  
####Known Issues:
* Based on speed of your processor build time may vary. By default *timeout* is set to "600000"ms in build.xml. Due to this you may face timeout issue. You can solve it by editing timeout to required value.

* If you get an OutofMemoryError during bootstrap, try setting environment variable ANT_OPTS to provide more memory, e.g.

  ```shell
  export ANT_OPTS="-Xmx1024m -Xms1024m"
  ```


###References:

http://maven.apache.org/ - The Maven website

http://maven.apache.org/guides/development/guide-building-maven.html - Further information on building ANTLR using Maven
