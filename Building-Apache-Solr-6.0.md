
**Apache Solr** can be built for Linux on z Systems running RHEL 7.1 or SLES 12 or Ubuntu 16.04 by following these instructions. Apache Solr version 6.0.0  has been successfully built & tested this way. More information on Apache Solr is available at http://lucene.apache.org/solr and the source code can be downloaded from http://apache.go-parts.com/lucene/solr/6.0.0

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
    
    For SLES 12
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
3. Download and Install ant 1.9.6 ( Only for RHEL 7.1 and SLES 12 )

	```
	wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.6-bin.tar.gz
	tar zxvf apache-ant-1.9.6-bin.tar.gz
	export PATH=/<source_root>/apache-ant-1.9.6/bin:$PATH
	```
	
### Product Build - Apache Solr

1. Download Apache Solr 6.0.0 source code

	```shell
	cd /<source_root>/
	wget http://apache.go-parts.com/lucene/solr/6.0.0/solr-6.0.0-src.tgz
	tar zxvf solr-6.0.0-src.tgz
	```
2. Configure and make

	For RHEL 7.1 and SLES 12 
    
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
	cd /<source_root>/solr-6.0.0/solr
	ant ivy-bootstrap
	ant server
	```
	_**Note:** messages such as `Execute failed: java.io.IOException: Cannot run program "svnversion"` are expected (and can safely be ignored) as we are using a snapshot of the Apache Solr code - not directly from the subversion repository. This command refers to adding version stamping to some files - you may see `${svnversion}` later on due to this but it is purely cosmetic._
3. Edit the solr and solr.cmd file (RHEL 7.1 and SLES 12 Only)

    (Only for IBM Java)
    ```shell
    cd /<source_root>/solr-6.0.0/solr/bin
    sed -i 's/Xloggc/Xverbosegclog/g' solr
    sed -i 's/JAVA_VERSION:(-2)/JAVA_VERSION:(-1)/g' solr
    sed -i 's/Xloggc/Xverbosegclog/g' solr.cmd
    ```
    The files solr and solr.cmd uses Xloggc which is not supported on IBM Java and hence we need to replace it with Xverbosegclog. Otherwise IBM java doesn't understand Xloggc and won't start. IBM java does not follow the version convention followed by Oracle and hence we get a syntax error while starting Solr, so we need to modify the IBM Java version check in solr script.

4. Verification

    ```shell
    cd /<source_root>/solr-6.0.0/solr/bin
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