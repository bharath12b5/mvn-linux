<!---PACKAGE:RabbitMQ--->
<!---DISTRO:SLES 12.x:3.6.1--->
<!---DISTRO:SLES 11.x:3.6.1--->
<!---DISTRO:RHEL 7.x:3.6.1--->
<!---DISTRO:RHEL 6.x:3.6.1--->
<!---DISTRO:Ubuntu 16.x:Distro,3.6.1--->

# Building RabbitMQ

Below version of RabbitMQ is available in respective distributions:

*    Ubuntu 16.04     has `3.5.7`

RabbitMQ version 3.6.1 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 6.7, RHEL 7.1/7.2, SLES 11-SP3, SLES 12/12-SP1 and Ubuntu 16.04.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

#### Step 1: Install the Dependencies
Required build/runtime dependencies:

* Install Erlang (For RHEL/SLES)
	
	Erlang > R13, See this article for complete build/install instructions: [Building Erlang on z](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang)

   
* RHEL7.1/7.2/6.7

			sudo yum install -y nc gzip findutils zip unzip libxslt xmlto patch subversion ca-certificates ant ant-junit java-1.7.1-ibm java-1.7.1-ibm-devel xz xz-devel git

* SLES12/12-SP1

			sudo zypper install -y zip unzip libxslt xmlto patch subversion procps ant ant-junit java-1_7_1-ibm java-1_7_1-ibm-devel git-core
			
* SLES11-SP3

			sudo zypper install -y zip unzip libxslt xmlto patch subversion procps ant ant-junit java-1_7_0-ibm java-1_7_0-ibm-devel python-devel python-xml  git-core

* Ubuntu 16.04

            sudo apt-get update
			sudo apt-get install ant openjdk-8-jdk erlang openssl wget tar xz-utils make python xsltproc rsync git zip

			
	**Note:-** Check the installed version of sed. If the sed version is below 4.2.2, use the following steps to build and install sed 4.2.2 from source:-
	
			cd /<source_root>/
			wget http://ftp.gnu.org/gnu/sed/sed-4.2.2.tar.gz
			tar -xvzf sed-4.2.2.tar.gz
			cd /<source_root>/sed-4.2.2
			./configure
			make
			sudo make install
			sudo cp /usr/local/bin/sed /usr/bin/sed
			
	**Note:-** Check the installed version of git. If the git version is <= 1.7.1, use the following steps to build and install higher version of git from source:-
	
			cd /<source_root>/
			sudo yum install -y curl-devel expat-devel gettext-devel openssl-devel perl-devel zlib-devel
			wget https://www.kernel.org/pub/software/scm/git/git-2.0.0.tar.gz
			tar xvzf git-2.0.0.tar.gz
			cd /<source_root>/git-2.0.0
			./configure --prefix=/usr
			make
			sudo make install
			
#### Step 2: Set environment variables
 Set JAVA_HOME, ANT_HOME, ERL_TOP and PATH.

**Note:** ERL_TOP is defined when installing Erlang in Step 1.

* RHEL7.1/7.2/RHEL6.7

			export JAVA_HOME=/usr/lib/jvm/java

* SLES12/12-SP1 and SLES11-SP3

			export JAVA_HOME=/usr/lib64/jvm/java
			
* RHEL7.1/7.2/RHEL6.7/SLES12/12-SP1/SLES11-SP3

			export ANT_HOME=/usr/share/ant
			export PATH=$PATH:$ERL_TOP/bin:/usr/lib/erlang/lib/erl_interface-3.7.20/bin/:$JAVA_HOME/bin:$ANT_HOME
			
#### Step 3: Download the source, build and test RabbitMQ Server 3.6.1
The server can be built from a standalone tarball. Alternatively the [RabbitMQ umbrella](https://www.rabbitmq.com/plugin-development.html) can be used for building the server. Using the umbrella is recommended if you intend to dabble in the RabbitMQ source or develop plugins for Rabbit. The sections below will cover building and installing the RabbitMQ Server 3.6.1 using both of the options.			
##### a. Download the source, build and test RabbitMQ Server 3.6.1

* Steps to build RabbitMQ server

			cd /<source_root>/
			wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1.tar.xz 
			tar xf rabbitmq-server-3.6.1.tar.xz
			cd /<source_root>/rabbitmq-server-3.6.1
			make all   

		
* Install the server, by setting the location of the RabbitMQ server binary, command and documentation respectively in the variables - `MAN_DIR, TARGET_DIR, SBIN_DIR`.   

			sudo TARGET_DIR=<dir> SBIN_DIR=<dir> MAN_DIR=<dir> make install   
			
	For example:
			
			sudo TARGET_DIR=/usr/bin SBIN_DIR=/usr/sbin MAN_DIR=/usr/share/man make install
			

##### b. Build, test and install the RabbitMQ Server 3.6.1 under the RabbitMQ Umbrella
* Download the umbrella and checkout the rabbitmq_v3_6_1 version
			
			cd /<source_root>/
		    git clone https://github.com/rabbitmq/rabbitmq-public-umbrella.git
			cd  /<source_root>/rabbitmq-public-umbrella 
			git checkout -b rabbitmq_v3_6_1
		
* Download all dependencies

			make co

* Download and extract the RabbitMQ Server 3.6.1
			
			cd /<source_root>/rabbitmq-public-umbrella/deps
			wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1.tar.xz 
			tar xf rabbitmq-server-3.6.1.tar.xz
			
* Build RabbitMQ server
			
			mv rabbitmq-server-3.6.1 rabbit
			cd /<source_root>/rabbitmq-public-umbrella/deps/rabbit
			make all

* Set environment variables for installation

			export TARGET_DIR=/usr/bin
			export SBIN_DIR=/usr/sbin
			export MAN_DIR=/usr/share/man

* Install RabbitMQ server

			sudo make install   
			

* (Optional) Test the RabbitMQ server with the Erlang client 3.6.1 test suite, Java client 3.6.1 test suite and Python QPid test suite. First, download the relevant packages for the 3.6.1 distro. The Makefile in the umbrella contains targets for downloading additional RabbitMQ repositories. Refer to the Makefile to download repositories as per your need.   

            # Download the clients for version 3.6.1
			cd /<source_root>/rabbitmq-public-umbrella/deps/
			wget https://www.rabbitmq.com/releases/rabbitmq-java-client/v3.6.1/rabbitmq-java-client-3.6.1.tar.gz 
			wget https://www.rabbitmq.com/releases/rabbitmq-erlang-client/v3.6.1/amqp_client-3.6.1-src.tar.xz 
			
            #Extract package and rename, so they can be used by other make targets			
			tar -xvzf rabbitmq-java-client-3.6.1.tar.gz 
			mv rabbitmq-java-client-3.6.1 rabbitmq_java_client 
			tar xf amqp_client-3.6.1-src.tar.xz 
			mv amqp_client-3.6.1-src amqp_client
			
			#Run the test suite	
			cd /<source_root>/rabbitmq-public-umbrella/deps/rabbitmq_test
			make all
			

#### Step 4: Set up RabbitMQ management UI and start RabbitMQ server

* Set environment variable for plugins

			export RABBITMQ_PLUGINS_DIR=/<source_root>/rabbitmq-public-umbrella/deps/rabbit/plugins

* Create directory for RabbitMQ

			sudo mkdir /etc/rabbitmq
			
	**Note:** RabbitMQ comes with default built-in settings which will be sufficient for running your RabbitMQ server effectively. 	
	In case you need to customize the settings for the RabbitMQ server, copy the _rabbitmq.config_ file in _/etc/rabbitmq_ directory.

* Enable RabbitMQ management plugin and start RabbitMQ server
	
			cd /<source_root>/rabbitmq-public-umbrella/deps/rabbit
			sudo make run-broker PLUGINS='rabbitmq_management rabbitmq_consistent_hash_exchange'
			
	RabbitMQ management UI is available at http://localhost:15672
			
   	
#### References:
https://github.com/rabbitmq/rabbitmq-public-umbrella.git
