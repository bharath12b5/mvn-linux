<!---PACKAGE:Apache Geode--->
<!---DISTRO:SLES 12:1.0.0--->
<!---DISTRO:RHEL 7.1:1.0.0--->
<!---DISTRO:Ubuntu 16.x:1.0.0--->

### Building Apache Geode

The instructions provided below specify the steps to build Apache Geode 1.0.0 on the IBM z Systems for following distributions:  
*	RHEL (6.8, 7.1, 7.2, 7.3)  
*	SLES (11 SP4, 12, 12 SP1, 12 SP2)  
*	Ubuntu (16.04, 16.10)  

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

##Step 1: Building and Installing Apache Geode
####1) Install dependencies  

  * RHEL 6.8
  
	With IBM SDK:
		
			sudo yum install -y git which java-1.8.0-ibm java-1.8.0-ibm-devel

  * RHEL (7.1, 7.2, 7.3)
  
    With IBM SDK:
		
			sudo yum install -y git which java-1.8.0-ibm java-1.8.0-ibm-devel
			
    With Open JDK:
		
			sudo yum install -y git which java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x	
  
  * SLES 11 SP4
  
	With IBM SDK:
			
			sudo zypper install -y git awk tar wget
			
	*  Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.
	
	_**Note:** Set JAVA_HOME to your defined java path. Also set JAVA_HOME/bin to PATH variable properly._
	
  * SLES 12
  
	With IBM SDK:
			
			sudo zypper install -y git which tar wget
			
	*  Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.
	
	_**Note:** Set JAVA_HOME to your defined java path. Also set JAVA_HOME/bin to PATH variable properly._
  
  * SLES (12 SP1, 12 SP2)
  
	With IBM SDK:
			
			sudo zypper install -y git which java-1_8_0-ibm java-1_8_0-ibm-devel
			
	With Open JDK:
		
			sudo zypper install -y git which java-1_8_0-openjdk java-1_8_0-openjdk-devel
		
  * Ubuntu (16.04, 16.10)
  
	With IBM SDK:
			
			sudo apt-get update
			sudo apt-get install -y git wget
	
	*  Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.
	
	_**Note:** Set JAVA_HOME to your defined java path. Also set JAVA_HOME/bin to PATH variable properly._

	With Open JDK:
		
			sudo apt-get update
			sudo apt-get install -y git openjdk-8-jdk
			

####2) Set Environment Variables(for Open JDK only)

  * RHEL (7.1, 7.2, 7.3)
  ```
  export JAVA_HOME=/usr/lib/jvm/java
  export LANG="en_US.UTF-8"
  export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
  export _JAVA_OPTIONS=-Xmx2048m
  export JVM_ARGS="-Xms2048m -Xmx2048m"
  ```  
    
  * SLES (12 SP1, SLES 12 SP2)
  ```
  export JAVA_HOME=/usr/lib64/jvm/java
  export LANG="en_US.UTF-8"
  export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
  export _JAVA_OPTIONS=-Xmx2048m
  export JVM_ARGS="-Xms2048m -Xmx2048m"
  ```

  * Ubuntu (16.04, 16.10)
  ```
  export LANG="en_US.UTF-8"
  export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
  export _JAVA_OPTIONS=-Xmx2048m
  export JVM_ARGS="-Xms2048m -Xmx2048m"
  ```

####3) Get the source
  ```
  cd /<source_root>/
  git clone https://github.com/apache/incubator-geode.git
  cd /<source_root>/incubator-geode/
  git checkout rel/v1.0.0-incubating
  ```

####4) Change gradle/wrapper/gradle-wrapper.properties to include gradle 3.1 binary (for IBM SDK only)  
```diff  
@@ -3,4 +3,4 @@ distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
-distributionUrl=https\://services.gradle.org/distributions/gradle-2.14.1-bin.zip
+distributionUrl=https\://services.gradle.org/distributions/gradle-3.1-bin.zip
```

####5) Build Apache Geode source without test cases  
  ```
  cd /<source_root>/incubator-geode/
  ./gradlew build installDist -x test
  ```
  
_**Note:** If build fails with `java.lang.StackOverflowError` error, set -Xss flag in gradle.properties file as follows:_

```diff
@@ -44,7 +44,7 @@ productName = Apache Geode (incubating)
productOrg = Apache Software Foundation (ASF)

org.gradle.daemon = true
org.gradle.jvmargs = -Xmx2048m
+org.gradle.jvmargs = -Xss<somevalue>m

```
	
##Step 6: Run test cases(Optional)  
  ```
  cd /<source_root>/incubator-geode/
  ./gradlew test
  ```  

_**Notes:**_
  * _With IBM SDK, there are few test case failures which are also observed on Intel x86 VM when built using IBM SDK. These failures can be ignored._  
  * _With Open JDK, there are few test case failures related to `CompressedOOPsObjectSize` flag which can be ignored as the basic functionality is not impacted._  
  * _Click [here](https://github.com/apache/incubator-geode/blob/rel/v1.0.0-incubating/README.md) to know more about how to start a locator and server. In case of "gfsh: command not found" error, set below path to PATH variable:_ 

  ```
  export PATH=$PATH:/<source_root>/incubator-geode/geode-assembly/build/install/apache-geode/bin
  ```

### References  
https://github.com/apache/incubator-geode  
https://cwiki.apache.org/confluence/display/GEODE/Index
http://geode.incubator.apache.org/
