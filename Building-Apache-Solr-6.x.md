
The instructions provided below specify the steps to build Apache Solr version 6.2.0 on Linux on the IBM z Systems for RHEL 7.1, SLES 12 SP1 and Ubuntu 16.04.

_**General Notes:**_ 	

i) _When following the steps below please use a standard permission user unless otherwise specified._
	 
ii) _A directory `/<source_root>/` will be referred to in these instructions. This is a temporary writable directory anywhere you'd like to place it._

## Building Apache Solr

### Obtain pre-built dependencies

1. Use the following commands to obtain dependencies

    For RHEL 7.1
    ```shell
    sudo yum install -y wget tar java-1.8.0-ibm-devel.s390x
    ```
    
    For SLES 12 SP1
    ```shell
    sudo zypper install -y wget tar java-1_8_0-ibm-devel
    ```

    For Ubuntu 16.04
    ```shell
    sudo apt-get install -y wget tar openjdk-8-jdk ant
    ```
2. Create the `/<source_root>/` as mentioned above

    ```shell
    mkdir /<source_root>/
    cd /<source_root>/
    ```
3. Download and Install ant 1.9.6 ( Only for RHEL 7.1 and SLES 12 SP1 )

	```
	wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.6-bin.tar.gz
	tar zxvf apache-ant-1.9.6-bin.tar.gz
	export PATH=/<source_root>/apache-ant-1.9.6/bin:$PATH
	```
	
### Product Build - Apache Solr

1. Download Apache Solr 6.2.0 source code

	```shell
	cd /<source_root>/
	wget http://archive.apache.org/dist/lucene/solr/6.2.0/solr-6.2.0-src.tgz
	tar zxvf solr-6.2.0-src.tgz
	```
2. Configure and make

	For RHEL7.1 and SLES12 SP1 
    
	If it throws error as JAVA_HOME is not defined correctly, export the JAVA_HOME as below
    ```shell
    export JAVA_HOME=/etc/alternatives/java_sdk_ibm
    ```
    For Ubuntu 16.04
	```shell
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
    ```
	
	Configure and make
    ```shell
	cd /<source_root>/solr-6.2.0/solr
	ant ivy-bootstrap
	ant server
	```

3. Verification

    ```shell
    cd /<source_root>/solr-6.2.0/solr/bin
    chmod a+x solr
    ./solr start
    ./solr create -c samp
    ```
    The solr start command will start the solr server on port 8983. solr create -c samp command would create the samp as
    ```
    {"responseHeader":{
    "status":0,
    "QTime":720},"core":"samp"}
    ```
    If the solr create command fails due to any reason, please stop and start the solr server and then rerun the command. 
	    
	After starting Solr, direct your Web browser to the Solr Admin Console at: http://HOST_IP:8983/solr/

###References:

http://lucene.apache.org/solr - Details of Apache Solr can be found here.
