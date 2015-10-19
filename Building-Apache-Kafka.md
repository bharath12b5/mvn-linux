**Apache Kafka** can be built for Linux on z Systems running RHEL 7.1 or 6.6 or SLES 12 or 11 by following these instructions.  Version 0.8.2.1 has been successfully built & tested this way.

_**General Notes:**_ 	
_i) When following the steps below please use a standard permission user unless otherwise specified._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_  
_iii) For convenience `vi` has been used in the instructions below when editing files, replace with your desired editing program if required_

## Building Apache Kafka

1. Install the build time dependencies

    On RHEL 7.1 and 6.6 systems
    ```shell
    sudo yum install git wget unzip java-1.7.1-ibm-devel
    ```
    On SLES 12 systems
    ```shell
    sudo zypper install git wget unzip java-1_7_1-ibm-devel
    ```
    On SLES 11 systems
    ```shell
    sudo zypper install git wget unzip java-1_7_0-ibm-devel
    ```
    You may already have some of these packages installed - just install any that are missing.  

3. Install gradle (unless you have it already)

    ```shell
    cd /<source_root>/
    wget https://services.gradle.org/distributions/gradle-2.5-bin.zip
    unzip gradle-2.5-bin.zip
    export PATH=$PATH:/<source_root>/gradle-2.5/bin
    ```
    Other versions of `gradle` may work, but this has been tested with `2.5`  
    _**Note:** Where `<source_root>` is the directory defined at the top of this document - if you wish to clean up the `<source_root>` at the end of the install you may want to place gradle in a different location, if so just update the paths_
4. Download the required version of the Apache Kafka source

    ```shell
    git clone https://github.com/apache/kafka
    cd kafka
    git checkout 0.8.2.1
    ```
    _**Note:** The github location is a mirror of the Apache location https://git-wip-us.apache.org/repos/asf/kafka.git_
5. Modify the build process to use a more recent build of Snappy Java

    Snappy Java prior to version `1.1.2` does not support Linux on z Systems, however fixes for the issue have been provided to the community and version `1.1.2` now incorporates an s390 build. However the current release (at the time of writing) of Apache Kafka does not use this later version of Snappy Java, so we update the build process to use it, as below:
    
    Modify the `build.gradle` file to use a later version
    ```shell
    vi build.gradle
    ```
    Update the snappy-java version to be `1.1.2` rather than `1.1.1.6` it is as default 
    ```gradle
    project(':clients') {
      archivesBaseName = "kafka-clients"
      
      dependencies {
        compile "org.slf4j:slf4j-api:1.7.6"
        compile 'org.xerial.snappy:snappy-java:1.1.2'
        compile 'net.jpountz.lz4:lz4:1.2.0'
    ```
   
6. Configure Java and run the gradle build process

    Set the `JAVA_HOME` environment variable to:
    ```shell
    export JAVA_HOME=/etc/alternatives/java_sdk_ibm
    ```
    Now setup gradle and build the jar files
    ```shell
    gradle
    gradle jar
    ```
    Multiple jar files are produced in multiple locations, most noteably the `clients`, `contrib` and `core` subdirectories, in which you'll find a path `build/libs/` which contain the jars
    ```shell
    ./examples/build/libs/kafka-examples-0.8.2.1.jar
    ./contrib/hadoop-consumer/build/libs/kafka-hadoop-consumer-0.8.2.1.jar
    ./contrib/build/libs/contrib-0.8.2.1.jar
    ./contrib/hadoop-producer/build/libs/kafka-hadoop-producer-0.8.2.1.jar
    ./core/build/libs/kafka_2.10-0.8.2.1.jar
    ./core/build/dependant-libs-2.10.4/zookeeper-3.4.6.jar
    ./core/build/dependant-libs-2.10.4/metrics-core-2.2.0.jar
    ./core/build/dependant-libs-2.10.4/snappy-java-1.1.2-SNAPSHOT.jar
    ./core/build/dependant-libs-2.10.4/slf4j-log4j12-1.7.6.jar
    ./core/build/dependant-libs-2.10.4/log4j-1.2.16.jar
    ./core/build/dependant-libs-2.10.4/scala-library-2.10.4.jar
    ./core/build/dependant-libs-2.10.4/lz4-1.2.0.jar
    ./core/build/dependant-libs-2.10.4/zkclient-0.3.jar
    ./core/build/dependant-libs-2.10.4/slf4j-api-1.7.6.jar
    ./core/build/dependant-libs-2.10.4/slf4j-log4j12-1.6.1.jar
    ./core/build/dependant-libs-2.10.4/jopt-simple-3.2.jar
    ./clients/build/libs/kafka-clients-0.8.2.1.jar
    ```

7. **Optionally** use the built in tests to verify Apache Kafka

    At the time of writing one of the scala test files didn't work with the IBM Java and so needs modifying
    ```shell
    vi /<source_root>/kafka/core/src/test/scala/other/kafka/TestOffsetManager.scala
    ```
    The issue is that the class being extended appears different under IBM's Java and the variable group needs to be renamed as follows:
    ```scala
    class CommitThread(id: Int, partitionCount: Int, commitIntervalMs: Long, zkClient: ZkClient)
          extends ShutdownableThread("commit-thread")
          with KafkaMetricsGroup {
        
      private val groupid = "group-" + id
      private val metadata = "Metadata from commit thread " + id
      private var offsetsChannel = ClientUtils.channelToOffsetManager(groupid, zkClient, SocketTimeoutMs)
    .....
      private def ensureConnected() {
        if (!offsetsChannel.isConnected)
          offsetsChannel = ClientUtils.channelToOffsetManager(groupid, zkClient, SocketTimeoutMs)
      }
    .....
     override def doWork() {
      val commitRequest = OffsetCommitRequest(groupid, immutable.Map((1 to partitionCount).map(TopicAndPartition("topic-" + id, _) -> OffsetAndMetadata(offset, metadata)):_*))
      try {
    .....
        case e2: IOException =>
          println("Commit thread %d: Error while committing offsets to %s:%d for group %s due to %s.".format(id, offsetsChannel.host, offsetsChannel.port, groupid, e2))
          offsetsChannel.disconnect()
    ```
    The variable `group` was renamed to `groupid` and all 5 instances were changed (shown above) - if any instances are missed the next step will fail, but correct the issue and re-run.
    ```shell
    gradle test
    ```
    We are currently expecting to see 2 failures out of 286 as follows:
    ```
    kafka.api.ProducerFailureHandlingTest > testNotEnoughReplicasAfterBrokerShutdown FAILED
        org.scalatest.junit.JUnitTestFailedError: Expected NotEnoughReplicasException when producing to topic with fewer brokers than min.insync.replicas
    
    kafka.admin.DeleteTopicTest > testDeleteTopicWithCleaner FAILED
        java.lang.OutOfMemoryError: Java heap space
    ```
    _**Note:** These two issues are not Linux on z Systems specific, they are also present on x86 platforms. Occasionally additional tests will fail, but re-running will normally resolve them - test stability issues have been reported [here](https://issues.apache.org/jira/browse/KAFKA-1970)_

8. **Optionally** use the example scripts to run a single server

    Apache Kafka provides a quickstart guide, this has a simple example that is replicated below (with a small patch), head to [kafka.apache.org](https://kafka.apache.org/08/quickstart.html) for the guide.
    
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