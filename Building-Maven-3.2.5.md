Maven can be built for Linux on z Systems running RHEL 7, RHEL 6, SLES 12 or SLES 11 by following these instructions.  Maven version 3.2.5 has been successfully built and tested this way.

_**General Notes:**_ 	

i) When following the steps below please use a standard permission user unless otherwise specified.
	 
ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

##Building Maven 3.2.5

1. Install build time dependencies

  For **SLES 12**
  ```shell
  sudo zypper install ant java-1_7_1-ibm-devel tar wget
  ```
  For **SLES 11**
  ```shell
  sudo zypper install java-1_7_0-ibm-devel tar wget       
  ```
  For **RHEL 7.1**
  ```shell
  sudo yum install ant java-1.7.1-ibm-devel tar wget
  ```
  For **RHEL 6.6**
  ```shell
  sudo yum install java-1.7.1-ibm-devel tar wget
  ```
 
  You may already have these packages installed - just install any missing.
  
  **Note:** Ant is not listed for SLES 11 or RHEL 6.6 as they provide an older version of ant than is required to build maven. Hence for SLES 11 and RHEL 6.6, ant is installed in a subsequent step below.
  
2. For **SLES 11 and RHEL 6.6** install appropriate version of ant 

  ```shell
  cd /<source_root>/
  wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.4-bin.tar.gz
  tar -zxvf apache-ant-1.9.4-bin.tar.gz
  export PATH=/<source_root>/apache-ant-1.9.4/bin:$PATH
  ```

3. Download the source code for maven and create a temporary installation directory

  ```shell
  cd /<source_root>/
  wget http://apache.cs.utah.edu/maven/maven-3/3.2.5/source/apache-maven-3.2.5-src.tar.gz
  tar -zxvf apache-maven-3.2.5-src.tar.gz
  mkdir /<source_root>/maven
  ```

4. Set environment variables

  For **SLES 12 and SLES 11**
  ```shell
  export JAVA_HOME=/usr/lib64/jvm/java
  ```
  For **RHEL 7.1 and RHEL 6.6**
  ```shell
  export JAVA_HOME=/usr/lib/jvm/java
  ```
  For **RHEL 7.1, RHEL 6.6, SLES 12 and SLES 11**
  ```shell
  export M2_HOME=/<source_root>/maven
  export PATH=$JAVA_HOME/bin:$PATH:$M2_HOME/bin 
  ```

5. Modify the build file

  Ant is used to build maven and it is necessary to modify the build file to turn off 'fork' for the 'maven-compile' target.
  The build file (build.xml) is located under `/<source_root>/apache-maven-3.2.5` and the following needs to be changed:

  Original line:
  ```xml
  <property name="maven-compile.fork" value="true"/>
  ```
  Needs to be changed to:
  ```xml
  <property name="maven-compile.fork" value="false"/>
  ```
  
6. Build Maven

  Ant needs to be invoked twice as shown below. A single invocation, with no target or target 'all', has been found to fail (believed due to required artifacts not being available to later stages of the build).

  ```shell
  cd /<source_root>/apache-maven-3.2.5
  ant maven-compile
  ant
  ```

  On completion of a successful build, a "BUILD SUCCESSFUL" message is output.
  
  Tests are run as part of the build and should all pass. Test reports can be found under:    
    `/<source_root>/apache-maven-3.2.5/maven-compat/target/surefire-reports`

7. Verification

  Run command `mvn --help` to verify installation.

8. Relocate the maven installation as necessary

  The above build creates a maven installation at $M2_HOME e.g. `/<source_root>/maven`. This may then be moved to an alternative location as desired.
  
  If the installation is moved then M2_HOME and PATH variables will need to be updated accordingly.
  

####Known Issues:
* Based on speed of your processor build time may vary. By default *timeout* is set to "600000"ms in build.xml. Due to this you may face timeout issue. You can solve it by editing timeout to required value.

* If you get an OutofMemoryError during bootstrap, try setting an environment variable MAVEN_OPTS to provide more memory, e.g.

        MAVEN_OPTS=-XX:MaxPermSize=128m -Xmx512m

##References:
http://maven.apache.org/