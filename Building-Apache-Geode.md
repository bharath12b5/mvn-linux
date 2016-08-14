<!---PACKAGE:Apache Geode--->
<!---DISTRO:SLES 12:1.0.0--->
<!---DISTRO:RHEL 7.1:1.0.0--->
<!---DISTRO:Ubuntu 16.x:1.0.0--->

### Building Apache Geode

Apache Geode 1.0.0 M1 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1, SLES 12 SP1 and Ubuntu 16.04.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

### Step 1: Install the Dependencies

For RHEL7.1:

  ```
  sudo yum install -y git which java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x
```       
            
For SLES12-SP1:

  ```    
  sudo zypper install -y git which java-1_8_0-openjdk java-1_8_0-openjdk-devel
```

For Ubuntu 16.04:  

  ```
  sudo apt-get install -y git openjdk-8-jdk
```

### Step 2: Set Environment Variables

For RHEL7.1:
  ```
  export JAVA_HOME=/usr/lib/jvm/java
  export LANG="en_US.UTF-8"
  export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
  export _JAVA_OPTIONS=-Xmx2048m
  export JVM_ARGS="-Xms2048m -Xmx2048m"
  ```  
    
For SLES12-SP1:
  ```
  export JAVA_HOME=/usr/lib64/jvm/java
  export LANG="en_US.UTF-8"
  export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
  export _JAVA_OPTIONS=-Xmx2048m
  export JVM_ARGS="-Xms2048m -Xmx2048m"
  ```

For Ubuntu 16.04:
  ```
  export LANG="en_US.UTF-8"
  export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF8"
  export _JAVA_OPTIONS=-Xmx2048m
  ```

### Step 3: Build and install Apache Geode
  1. Get the source
     ```
	 
		cd /<source_root>/
		git clone https://github.com/apache/incubator-geode.git
		cd /<source_root>/incubator-geode/
		git checkout rel/v1.0.0-incubating.M1
    ```
  2. Change gemfire-core/build.gradle to include snappy-java-1.1.2.jar:

    ```
		vi /<source_root>/incubator-geode/gemfire-core/build.gradle
    ```
    In the dependencies section, change the snappy-java entry as follows:

    ```
		dependencies {
		....
			compile 'org.xerial.snappy:snappy-java:1.1.2'
		}
    ``` 
 
  3. Build Apache Geode source without test cases  
    ```
		./gradlew build installDist -x test
  ```

  4. Run test cases(Optional)  
    ```
		./gradlew test
  ```

_**Note:** Click [here](https://github.com/apache/incubator-geode/blob/rel/v1.0.0-incubating.M1/README.md) to know more about how to start a locator and server._

### References  
https://github.com/apache/incubator-geode  
https://cwiki.apache.org/confluence/display/GEODE/Index
http://geode.incubator.apache.org/
