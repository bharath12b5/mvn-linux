<!---PACKAGE:Chronos--->
<!---DISTRO:SLES 12:2.4.0--->
<!---DISTRO:RHEL 7.1:2.4.0--->
<!---DISTRO:Ubuntu 16.04:2.4.0--->

# Building Chronos
[Chronos](https://mesos.github.io/chronos/docs/) is a replacement for cron. It is a distributed and fault-tolerant scheduler that runs on top of Apache Mesos that can be used for job orchestration. The stable release of Chronos 2.4.0 has been built and tested on Linux on z Systems. The following instructions can be used for RHEL 7.1, SLES 12 and Ubuntu 16.04.

#### General Notes: ####
i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it.

### Prerequisites: ###
*    **Maven v3.0.0** or above -- Instructions for building Maven can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven).

    _**Note:**_ Maven will be used in Step 2 below.

*     **Apache Zookeeper** -- Instructions for building Zookeeper can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-ZooKeeper).

    _**Note:**_ Zookeeper will be used in Step 4 below.

*     **Apache Mesos 0.28.0** or above -- Instructions for building Mesos can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-Mesos).

    _**Note:**_ Mesos will be used in Step 5 below.
   
## Building and Installing Chronos

### 1. Install the Dependencies

For RHEL 7.1

```
sudo yum install wget tar git gcc gcc-c++ make java-1.8.0-openjdk
```

For SLES 12

```
sudo zypper install wget tar git gcc gcc-c++ make java-1_8_0-openjdk
```

For Ubuntu 16.04

```
sudo apt-get install wget tar git gcc g++ make openjdk-8-jdk
```

### 2. Set environment variables for MAVEN

```
export MAVEN_HOME=/<source_root>/apache-maven-3.3.9
```

### 3. Build and install Node

```
cd /<source_root>/
git clone https://github.com/andrewlow/node.git
cd node/
./configure
make
sudo make install
```

### 4. Start Zookeeper by doing the following:
To start ZooKeeper you need a configuration file. Here is a sample, create it in conf/zoo.cfg:

```
cd /<source_root>/zookeeper/
vi conf/zoo.cfg
```

Add the below contents to the file:
```
# The number of milliseconds of each tick
tickTime=2000
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
```

Start ZooKeeper
```
bin/zkServer.sh start
```
The binary will be available at `/<source_root>/zookeeper/bin`

_**Note:**_ For Ubuntu 16.04 systems, if zookeeper is installed from the repo, the above steps need not be performed to start zookeeper. Just start zookeeper as a service and verify that the zoo.cfg file has the above contents set by default, if not please add the above mentioned contents.

### 5. Start Mesos master and slave(s)
```
cd /<source_root>/mesos/build
# Start Mesos master (Ensure work directory exists and has proper permissions).
$ ./bin/mesos-master.sh --ip=127.0.0.1 --work_dir=/var/lib/mesos

# Start Mesos slave.
$ ./bin/mesos-slave.sh --master=127.0.0.1:5050
```

### 6. Build Chronos
```
export MESOS_NATIVE_LIBRARY=/usr/local/lib/libmesos.so
cd /<source_root>/
git clone https://github.com/mesos/chronos.git
cd chronos
mvn package
bin/start-chronos.bash --http_port 4400
```

### 7. Verify Chronos is running successfully

Open the Chronos web interface(http://localhost:4400),

1. Click New Job.

2. Add the NAME "Test Chronos Sleep", COMMAND "sleep 10", your email for OWNER(S), and click Create.

3. After the job is created, you can click on Force Run (the 'play' button) to run it.

You should see it as a running task on your Mesos master and the Sandbox. Once it's finished running, you should see status “SUCCESS.”


## References:

*    https://mesos.github.io/chronos/docs/getting-started.html 
*    https://docs.mesosphere.com/1.7/usage/services/chronos/
*    https://zookeeper.apache.org/doc/r3.5.1-alpha/zookeeperStarted.html 

