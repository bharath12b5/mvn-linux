# Building Elasticsearch

The instructions provided below specify the steps to build [Elasticsearch](https://www.elastic.co/products/elasticsearch) v1.5.1 & v2.1.0 on Linux on the IBM z Systems for RHEL 7 and SLES 12.

NOTE: When following the steps below, please use a standard permission user unless otherwise specified.

### Install OpenJDK
#### RHEL7
        sudo yum install java-1.8.0-openjdk.s390x
        sudo yum install java-1.8.0-openjdk-devel.s390x

#### SLES12
        sudo zypper install java-1_7_0-openjdk
        sudo zypper install java-1_7_0-openjdk-devel

#### Download Maven and add it to the path. 
Download the "apache-maven-3.3.3-bin.tar.gz" from: https://maven.apache.org/download.cgi. Alternatively you can build your own by following these instructions: 
https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven

        tar -zxvf apache-maven-3.3.3-bin.tar.gz
In the ˜/.bashrc file add.
       
        MAVEN="/<MAVEN LOCATION>/apache-maven-3.3.3/bin"
        PATH=$PATH:$MAVEN
        export PATH

Increase the maximum number of files that are allowed to be opened on the system. 

"ulimit -n" should be greater than 32k or 64k. 32k and 64k are the suggested sizes.

In order to increase this limit, follow the given steps.
 
As root, edit the file /etc/security/limits.conf, add the given line.

        *       -       nofile      65535

Also as root, run the command "ulimit -n 65535". 
Logout and Login. Run the command ulimit -n, it should return 65535.


#### Obtain required version of Elasticsearch

       wget https://github.com/elastic/elasticsearch/archive/v1.5.1.zip
       unzip v1.5.1.zip


##### Add the Sigar library
Add the correct sigar library ".so" files to the <elasticsearch-directory>/lib/sigar.

The one for s390x can be obtained from the following page: http://sourceforge.net/projects/sigar/files/sigar/1.6/ (As suggested on: 
https://support.hyperic.com/display/SIGAR/Home#Home-download )

Unzip the downloaded folder and obtain the specific one for s390x (libsigar-s390x-linux.so) library from under hyperic-sigar-<version>\hyperic-sigar-<version>\sigar-bin\lib. This file needs to be added to <elasticsearch dir>/lib/sigar.


#### Build the package
To build the package without the tests: 

      mvn clean package -DskipTests

To build with local tests: 

        mvn clean package -Des.node.mode=local

To build with the network tests: 

        mvn clean package -Des.node.mode=network

The default build (mvn clean package) without any parameters is -Des.node.mode=local 
Elasticseach uses randomized testing, and hence many tests might fail unpredictably, this can be ignored. If the -DskipTests succeeds the build is successful