#Building AntLR 3.5.2
Below versions of AntLR are available in respective distributions at the time of this recipe creation:

*	RHEL 6.7 has `2.7.7-6.5.el6`
*	RHEL 7.1/7.2 has `	2.7.7-30.el7`
*	SLES 11-SP3 has `2.7.7-1.27`
*	SLES 12/12-SP1 has `2.7.7-98.147`
*	Ubuntu 16.04 has `3.5.2`

The instructions provided below specify the steps to build [AntLR](http://www.antlr.org/) 3.5.2 on Linux on the IBM z Systems for RHEL 6.7/7.1/7.2 and SLES 11-SP3/12/12-SP1.

_**General Notes:**_ 	

i) When following the steps below please use a standard permission user unless otherwise specified.
	 
ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

## Building ANTLR

### Obtain pre-built dependencies

1. Use the following commands to obtain dependencies

    For RHEL 7.1/7.2

  ```shell
  sudo yum install -y git java-1.7.1-ibm-devel tar
  ```
	
    For RHEL 6.7

  ```shell
  sudo yum install -y git java-1.7.1-ibm-devel tar
  ```
	
    For SLES 12/12-SP1

  ```shell
  sudo zypper install -y git java-1_7_1-ibm-devel tar 
  ```
	
    For SLES 11-SP3

  ```shell
  sudo zypper install -y git java-1_7_0-ibm-devel tar
  ```

### Dependency Build - Maven 3.2.5

  This recipe uses Maven to build ANTLR so it is first necessary to build Maven. Full instructions can be found at [Building-Maven](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven).
  
### Product Build - ANTLR

1. Set environment variables and working directory

  For **SLES 11-SP3 and SLES 12/12-SP1**
  
  ```shell
  export JAVA_HOME=/usr/lib64/jvm/java
  ```
  
  For **RHEL 6.7 and RHEL 7.1/7.2**
  
  ```shell
  export JAVA_HOME=/usr/lib/jvm/java
  ```
  
  For **RHEL 6.7, RHEL 7.1/7.2 , SLES 11-SP3 and SLES 12/12-SP1**
  
  ```shell
  cd /<source_root>/
  export WORK_DIR=`pwd`
  export M2_HOME=$WORK_DIR/maven
  export PATH=$JAVA_HOME/bin:$PATH:$M2_HOME/bin
  export CLASSPATH=$WORK_DIR/antlr3/antlr-complete/target/antlr-complete-3.5.2.jar:$CLASSPATH
  ```
	
1. Download the ANTLR source code

  ```shell
  cd $WORK_DIR
  git clone https://github.com/antlr/antlr3
  cd antlr3
  git checkout 3.5.2
  ```
	
1. Build and test ANTLR

    ```shell
    cd $WORK_DIR/antlr3
    mvn -Dgpg.skip=true -Duser.name="test" -Dbootclasspath.compile=$JAVA_HOME/jre/lib/s390x/default/jclSC170/vm.jar:$JAVA_HOME/jre/lib/rt.jar -Dbootclasspath.testCompile=$JAVA_HOME/jre/lib/s390x/default/jclSC170/vm.jar:$JAVA_HOME/jre/lib/rt.jar -Djava6.home=$JAVA_HOME/jre install | tee antlr_build_log
    ```

  Note that test results are included in the output of the above command, hence we capture the output in antlr_build_log. Relevant lines contain the text "Tests run". There should be no test failures/errors.


1. Verify the build

  The following command should display the version of ANTLR that has been built and output usage information.

  ```shell
  java org.antlr.Tool
  ```

1. Install ANTLR

  To install ANTLR, copy antlr-complete-3.5.2.jar from $WORK_DIR/antlr3/antlr-complete/target to your desired installation location and update environment variable CLASSPATH accordingly.

  The installation may be verified by running the same command that was used to verify the build (in the previous step).
	

1. _[Optional]_ Clean up

  Once you have a copy of antlr-complete-3.5.2.jar outside $WORK_DIR then $WORK_DIR may be deleted.
  

###References:

http://www.antlr.org/ - The ANTLR website

https://theantlrguy.atlassian.net/wiki/display/ANTLR3/Building+ANTLR+with+Maven - Further information on building ANTLR using Maven
