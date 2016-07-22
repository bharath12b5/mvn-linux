<!---PACKAGE:Apache Kafka--->
<!---DISTRO:SLES 12:0.10.0--->
<!---DISTRO:SLES 11:0.10.0--->
<!---DISTRO:RHEL 7.1:0.10.0--->
<!---DISTRO:RHEL 6.6:0.10.0--->
<!---DISTRO:Ubuntu 16.x:0.10.0--->

The instructions provided below specify the steps to build Apache Kafka 0.10.0.0 on Linux on the IBM z Systems for RHEL 6/7, SLES 11/12 and Ubuntu 16.04.

_**General Notes:**_ 	
_i) When following the steps below please use a standard permission user unless otherwise specified._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_  
_iii) For convenience `vi` has been used in the instructions below when editing files, replace with your desired editing program if required_


## Building Apache Kafka

1. Install the dependencies

    On RHEL 6.6 systems
	 ```shell
     sudo yum install git wget unzip java-1.7.1-ibm-devel.s390x
     ```
	
	RHEL 7.1 systems
	
	 * With IBM JDK  
      ```shell
      sudo yum install git wget unzip java-1.7.1-ibm-devel.s390x
      ```
	
	 * With OpenJDK 
      ```shell
      sudo yum install git wget unzip java-1.8.0-openjdk-devel.s390x
      ```
    
	On SLES 12 systems
    
	 * With IBM JDK
	  ```shell
      sudo zypper install git wget unzip java-1_7_1-ibm-devel
      ```
	
	 * With OpenJDK 
      ```shell
      sudo zypper install git wget unzip java-1_7_0-openjdk-devel
      ```
    
	On SLES 11 systems
	 ```shell
     sudo zypper install git wget unzip java-1_7_0-ibm-devel java-1_7_0-ibm
     ```
    
	On Ubuntu 16.04 systems
    
	 * With IBM JDK
	
	  Install IBM Java 8
	
	  Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link
	
	  ```shell
      sudo apt-get update
      sudo apt-get install git wget unzip
      ```
	
	 * With OpenJDK
	  ```shell
      sudo apt-get update
      sudo apt-get install git wget unzip openjdk-8-jdk
      ```
	
    You may already have some of these packages installed - just install any that are missing.  

2. Install gradle (unless you have it already)

    ```shell
    cd /<source_root>/
    wget https://services.gradle.org/distributions/gradle-2.5-bin.zip
    unzip gradle-2.5-bin.zip
    export PATH=$PATH:/<source_root>/gradle-2.5/bin
    ```
    Other versions of `gradle` may work, but this has been tested with `2.5`  
    _**Note:** Where `<source_root>` is the directory defined at the top of this document - if you wish to clean up the `<source_root>` at the end of the install you may want to place gradle in a different location, if so just update the paths_

3. Download the required version of the Apache Kafka source

    ```shell
    git clone https://github.com/apache/kafka
    cd kafka
    git checkout 0.10.0
    ```
    _**Note:** The github location is a mirror of the Apache location https://git-wip-us.apache.org/repos/asf/kafka.git_
   
4. Configure Java and run the gradle build process

   Set the `JAVA_HOME` environment variable to:
        
   For RHEL/SLES
     
	 * With IBM JDK
	 ```shell
      export JAVA_HOME=/etc/alternatives/java_sdk_ibm
      ```
	 
	 * With OpenJDK
      ```shell

      export JAVA_HOME=/etc/alternatives/java_sdk_openjdk

      ```        
   
   For Ubuntu (Only with IBM JDK)
      ```shell
      export JAVA_HOME=/<source_root>/ibm-java-s390x-80
      ```
   _**Note:** Where `<source_root>` is the directory defined at the top of this document - if you wish to clean up the `<source_root>` at the end of the install you may want to place ibm-java-s390x-80 in a different location, if so just update the JAVA_HOME_

    Now setup gradle and build the jar files

	  ```
      gradle
      gradle jar
	  ```
5. **Optionally** use the built in tests to verify Apache Kafka

    ```
    gradle test
    ```

    _**Note:** Occasionally tests will fail,but re-running will normally resolve them - These issues are not Linux on z Systems specific, they are also present on x86 platforms. Test stability issues have been reported [here](https://issues.apache.org/jira/browse/KAFKA-1970)_

6. **Optionally** use the example scripts to run a single server

    Apache Kafka provides a quickstart guide, this has a simple example that is replicated below (with a small patch), head to [kafka.apache.org](https://kafka.apache.org/08/quickstart.html) for the guide.
    
	1. Update the following files
	
	 * **With IBM JDK**
     
	  1. Update the `-X` options
    
        ```shell
        cd /<source_root>/kafka/
        vi bin/kafka-run-class.sh
        ```
        The script that starts the zookeeper creates a gc log, however it uses an option that isn't supported on IBM Java (the -X options are VM specific extensions to the Java spec), so we need to update `-Xloggc` to `-Xverbosegclog` as below:
        ```shell
        # GC options
        GC_FILE_SUFFIX='-gc.log'
        GC_LOG_FILE_NAME=''
        if [ "x$GC_LOG_ENABLED" = "xtrue" ]; then
          GC_LOG_FILE_NAME=$DAEMON_NAME$GC_FILE_SUFFIX
          KAFKA_GC_LOG_OPTS="-Xverbosegclog:$LOG_DIR/$GC_LOG_FILE_NAME -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps "
        fi
        ```
	
	  2. Remove `-loggc` options to update the script to start the servers. (on Ubuntu 16.04 **only**) 
		
		 * Update zookeeper-server-start.sh
			
			```shell
			cd /<source_root>/kafka
			vi bin/zookeeper-server-start.sh
			```
			The script that starts the zookeeper creates a gc log, however it uses an option that isn't supported on IBM Java, so we need to remove `-loggc` as below:
			```shell
			EXTRA_ARGS="-name zookeeper"
			```
				
		 * Update kafka-server-start.sh
		
			```shell
			cd /<source_root>/kafka
			vi bin/kafka-server-start.sh
			```
			The script that starts the kafka creates a gc log, however it uses an option that isn't supported on IBM Java, so we need to remove `-loggc` as below:
			```shell
			EXTRA_ARGS="-name kafkaServer"
			```
	
	 * **With OpenJDK**
	
      1. Remove the `-XX:+UseG1GC` options
    
        ```shell
        cd /<source_root>/kafka/
        vi bin/kafka-run-class.sh
        ```
        The script that starts the zookeeper uses Garbage-First Collector, however it isn't supported on System Z, so we need to remove `-XX:+UseG1GC` as below:
        ```shell
		# JVM performance options
		if [ -z "$KAFKA_JVM_PERFORMANCE_OPTS" ]; then
		  KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+DisableExplicitGC -Djava.awt.headless=true"
		fi
		```

	2. Now launch the two single node servers
				
		```shell
		bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
		bin/kafka-server-start.sh -daemon config/server.properties
		```
		
		Both servers should start in the background, check the logs in `/<source_root>/kafka/logs/` for more information.  
		
    3. Now create and list a simple test topic and messages file
    
		 Please note the quickstart guide refer to `bin/kafka-create-topic.sh` however this has been replaced with `bin/kafka-topics.sh` and flags such as `--create` or `--delete` and some modifications to the parameters:

		```shell
		bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partition 1 --topic test
		bin/kafka-topics.sh --list --zookeeper localhost:2181
		```

		 Create a quick messages file (you can also enter it on the console) then produce those messages onto the test topic created earlier

		```shell
		echo -e "Congratulations\nThe build is working\n\nWelcome to Apache Kafka with Linux on z Systems" > /tmp/msg.log
		bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test < /tmp/msg.log
		```

	4. Run the consumer and check the results
    
		And finally run a consumer to pull these messages back off the node

		```shell
		bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning --max-messages 4
		```

		You should see the following:

		```shell
		Congratulations
		The build is working
			
		Welcome to Apache Kafka with Linux on z Systems
		Consumed 4 messages
		```
		
		_**Note:** If the producer is run to accept input from the console (without the `< /tmp/msg.log` part) and the consumer is run without `--max-messages 4` from two different terminals you can see response as you enter each line_  

### References
http://kafka.apache.org/  
https://github.com/apache/kafka  
https://kafka.apache.org/08/quickstart.html  
