<!---PACKAGE:Apache Kafka--->
<!---DISTRO:SLES 12.x:0.10.1--->
<!---DISTRO:SLES 11.x:0.10.1--->
<!---DISTRO:RHEL 7.x:0.10.1--->
<!---DISTRO:RHEL 6.x:0.10.1--->
<!---DISTRO:Ubuntu 16.x:0.10.1--->

# Building Apache Kafka

The instructions provided below specify the steps to build Apache Kafka 0.10.1.0 on IBM z Systems for RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	
* _When following the steps below please use a standard permission user unless otherwise specified._  
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._  

##Step 1: Building and installing Apache Kafka

####1.1) Install dependencies

  * RHEL 6.8
    ```bash
    sudo yum install git wget unzip java-1.7.1-ibm-devel
    ```
	
  * RHEL 7.1/7.2/7.3
	
    * With IBM JDK  
      ```bash
      sudo yum install git wget unzip java-1.8.0-ibm-devel
      ```
	
    * With OpenJDK 
      ```bash
      sudo yum install git wget unzip java-1.8.0-openjdk-devel
      ```
    
  * SLES 11-SP4
    ```bash
    sudo zypper install git wget unzip java-1_7_0-ibm-devel java-1_7_0-ibm
    ```
     
  * SLES 12
    
    * With IBM JDK
      ```bash
      sudo zypper install git wget unzip java-1_7_1-ibm-devel
      ```
	
    * With OpenJDK 
      ```bash
      sudo zypper install git wget unzip java-1_7_0-openjdk-devel
      ```
    
  * SLES 12-SP1/12-SP2
    
    * With IBM JDK
      ```bash
      sudo zypper install git wget unzip java-1_8_0-ibm-devel
      ```
	
    * With OpenJDK 
      ```bash
      sudo zypper install git wget unzip java-1_8_0-openjdk-devel
      ```

  * Ubuntu 16.04/16.10
    
    * With IBM JDK
	
      ```bash
      sudo apt-get update
      sudo apt-get install git wget unzip
      ```
		
      Install IBM Java 8
	
      Download IBM Java 8 SDK binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per the given link.

    * With OpenJDK
      ```bash
      sudo apt-get update
      sudo apt-get install git wget unzip openjdk-8-jdk
      ```

####1.2) Install Gradle

  ```bash
  cd /<source_root>/
  wget https://services.gradle.org/distributions/gradle-3.1-bin.zip
  unzip gradle-3.1-bin.zip
  ```

####1.3) Download the source code

  ```bash
  git clone https://github.com/apache/kafka
  cd kafka
  git checkout 0.10.1.0
  ```
   
####1.4) Set environment variables

* Update `PATH` variable
    ```bash
    export PATH=$PATH:/<source_root>/gradle-3.1/bin
    ```

* Set the `JAVA_HOME`variable
        
  * RHEL
    ```bash
    export JAVA_HOME=/usr/lib/jvm/java
    ```        
 
  * SLES
    ```bash
    export JAVA_HOME=/usr/lib64/jvm/java
    ``` 

  * Ubuntu
    
	* With IBM JDK
      ```bash
      export JAVA_HOME=/<user_install_dir>/ibm/java
      ```
      _**Note:** Where `/<user_install_dir>/` is the location where IBM SKD is installed. Ideally the location is `/opt`._
	 
    * With OpenJDK
      ```bash
      export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
      ```        
  
    
####1.5) Setup Gradle and build the jar files

  ```bash
  gradle
  gradle jar
  ```

##Step 2: Testing (Optional)

  ```bash
  gradle test
  ```

_**Note:** Occasionally tests will fail, but re-running will normally resolve them._

##Step 3: Use the example scripts to run a single server (Optional)

Apache Kafka provides a quick-start guide, this has a simple example that is replicated below (with a small patch), head to [kafka.apache.org](https://kafka.apache.org/quickstart) for the guide.

####3.1) Update the following files
	
* With IBM JDK
     
  * Update `/<source_root>/kafka/bin/kafka-run-class.sh`
    
    ```diff
    @@ -245,7 +245,7 @@ GC_FILE_SUFFIX='-gc.log'
      GC_LOG_FILE_NAME=''
      if [ "x$GC_LOG_ENABLED" = "xtrue" ]; then
        GC_LOG_FILE_NAME=$DAEMON_NAME$GC_FILE_SUFFIX
     -  KAFKA_GC_LOG_OPTS="-Xloggc:$LOG_DIR/$GC_LOG_FILE_NAME -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps "
     +  KAFKA_GC_LOG_OPTS="-Xverbosegclog:$LOG_DIR/$GC_LOG_FILE_NAME -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps "
      fi

     # If Cygwin is detected, classpath is converted to Windows format.
     ```
      
  * Update `/<source_root>/kafka/zookeeper-server-start.sh` (**Only** for Ubuntu)
		
    ```diff
    @@ -29,7 +29,7 @@ if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
          export KAFKA_HEAP_OPTS="-Xmx512M -Xms512M"
      fi
         
     -EXTRA_ARGS=${EXTRA_ARGS-'-name zookeeper -loggc'}
     +EXTRA_ARGS=${EXTRA_ARGS-'-name zookeeper'}
      
      COMMAND=$1
      case $COMMAND in
    ```
		
  * Update `/<source_root>/kafka/bin/kafka-server-start.sh` (**Only** for Ubuntu)
        
    ```diff
    @@ -29,7 +29,7 @@ if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
          export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
      fi
  
     -EXTRA_ARGS=${EXTRA_ARGS-'-name kafkaServer -loggc'}
     +EXTRA_ARGS=${EXTRA_ARGS-'-name kafkaServer'}
  
      COMMAND=$1
      case $COMMAND in
    ```		
	
* With OpenJDK
	
  * Update `/<source_root>/kafka/bin/kafka-run-class.sh`
    
    ```diff
	@@ -212,7 +212,7 @@ fi
  
	  # JVM performance options
	  if [ -z "$KAFKA_JVM_PERFORMANCE_OPTS" ]; then
	 -  KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+DisableExplicitGC -Djava.awt.headless=true"
	 +  KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+DisableExplicitGC -Djava.awt.headless=true"
	  fi
  
  
	```

####3.2) Now launch the two single node servers
				
  ```bash
  bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
  bin/kafka-server-start.sh -daemon config/server.properties
  ```
		
_**Note:** Both servers should start in the background, check the logs in `/<source_root>/kafka/logs/` for more information._
		
####3.3) Now create and list a simple test topic and messages file

  ```bash
  bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partition 1 --topic test
  bin/kafka-topics.sh --list --zookeeper localhost:2181
  ```

  Create a quick messages file (you can also enter it on the console) then produce those messages onto the test topic created earlier

  ```bash
  echo -e "Congratulations\nThe build is working\n\nWelcome to Apache Kafka with Linux on z Systems" > /tmp/msg.log
  bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test < /tmp/msg.log
  ```

####3.4) Run the consumer and check the results

  And finally run a consumer to pull these messages back off the node

  ```bash
  bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning --max-messages 4
  ```

  You should see the following:

  ```bash
  Congratulations
  The build is working
			
  Welcome to Apache Kafka with Linux on z Systems
  Consumed 4 messages
  ```
		
_**Note:** If the producer is run to accept input from the console (without the `< /tmp/msg.log` part) and the consumer is run without `--max-messages 4` from two different terminals you can see response as you enter each line._  

## References:
http://kafka.apache.org/  
https://kafka.apache.org/quickstart
