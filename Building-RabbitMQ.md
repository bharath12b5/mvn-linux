<!---PACKAGE:RabbitMQ--->
<!---DISTRO:SLES 12.x:3.6.6--->
<!---DISTRO:SLES 11.x:3.6.6--->
<!---DISTRO:RHEL 7.x:3.6.6--->
<!---DISTRO:RHEL 6.x:3.6.6--->
<!---DISTRO:Ubuntu 16.x:Distro,3.6.6--->

# Building RabbitMQ

Below version of RabbitMQ is available in respective distributions:

* Ubuntu 16.04 and 16.10 have `3.5.7`

The instructions provided below specify the steps to build RabbitMQ version 3.6.6 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.  

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##Step 1: Building and installing RabbitMQ
####1.1) Install dependencies

* Install Erlang (for RHEL and SLES distributions only)
	
  See this article for complete build/install instructions: [Building Erlang on z](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang).

   
* RHEL 6.8 and RHEL 7.1/7.2/7.3
  ```sh
  sudo yum install c gzip findutils zip unzip libxslt xmlto patch subversion ca-certificates ant ant-junit java-1.7.1-ibm java-1.7.1-ibm-devel xz xz-devel git wget tar make
  ```

* SLES 11-SP4
  ```sh
  sudo zypper install zip unzip libxslt xmlto patch subversion procps ant ant-junit java-1_7_0-ibm java-1_7_0-ibm-devel python-devel python-xml  git-core xz
  ```

* SLES 12/12-SP1/12-SP2
  ```sh
  sudo zypper install zip unzip libxslt xmlto patch subversion procps ant ant-junit java-1_7_1-ibm java-1_7_1-ibm-devel git-core
  ```
  
* Ubuntu 16.04/16.10
  ```sh
  sudo apt-get update
  sudo apt-get install ant openjdk-8-jdk erlang openssl wget tar xz-utils make python xsltproc rsync git zip
  ```
  
 _**Note:** Check the installed version of `sed`. If the `sed` version is below `4.2.2`, use the following steps to build and install `sed 4.2.2` from source:_
  ```sh
  cd /<source_root>/
  wget http://ftp.gnu.org/gnu/sed/sed-4.2.2.tar.gz
  tar -xvzf sed-4.2.2.tar.gz
  cd /<source_root>/sed-4.2.2
  ./configure
  make
  sudo make install
  sudo cp /usr/local/bin/sed /usr/bin/sed
  ```

####1.2) Set environment variables (for RHEL and SLES distributions only)
* Set `JAVA_HOME`
  ```sh
  export JAVA_HOME=/usr/lib/jvm/java #only for RHEL distributions
  export JAVA_HOME=/usr/lib64/jvm/java #only for SLES distributions
  ```
  
* Set `ANT_HOME`
  ```sh
  export ANT_HOME=/usr/share/ant
  ```

* Set `PATH`
  ```sh
  export PATH=$PATH:$ERL_TOP/bin:/usr/lib/erlang/lib/erl_interface-3.9/bin/:$JAVA_HOME/bin:$ANT_HOME
  ```

_**Note:** `ERL_TOP` is defined when installing Erlang._

####1.3) Download the source and build RabbitMQ Server
  ```sh
  cd /<source_root>/
  wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.6/rabbitmq-server-3.6.6.tar.xz 
  tar xf rabbitmq-server-3.6.6.tar.xz
  cd /<source_root>/rabbitmq-server-3.6.6
  make all
  sudo make install
  ```
  
##Step 2: Testing(Optional)

####2.1) Test the RabbitMQ server
  ```sh
  cd /<source_root>/rabbitmq-server-3.6.6
  sudo make tests
  ```
  _**Note:** Instructions to run full server tests, integration tests and unit tests can be found [here](https://github.com/rabbitmq/rabbitmq-public-umbrella/tree/rabbitmq_v3_6_6#running-tests)._

##Step 3: Start RabbitMQ server(Optional)

####3.1) Build and install Elixir
  ```sh
  cd /<source_root>/
  git clone https://github.com/elixir-lang/elixir
  cd /<source_root>/elixir
  git checkout v1.3.4
  make
  sudo make install
  ```

####3.2) Running RabbitMQ from source(Optional)
  ```sh
  cd /<source_root>/rabbitmq-server-3.6.6
  sudo make run-broker 
  ```

  _**Note:** RabbitMQ management UI is available at `http://localhost:15672`._

####3.3) Running RabbitMQ from script(Optional)
  * Create directory for RabbitMQ
    ```sh
    sudo mkdir /etc/rabbitmq
    ```

  _**Note:** RabbitMQ comes with default built-in settings which will be sufficient for running your RabbitMQ server effectively. In case you need to customize the settings for the RabbitMQ server, copy the `rabbitmq.config` file in `/etc/rabbitmq` directory._

  * Enable RabbitMQ management plugin and start RabbitMQ server
    ```sh
    cd /<source_root>/rabbitmq-server-3.6.6
    sudo ln -s $PWD/plugins deps/rabbit/plugins
    sudo deps/rabbit/scripts/rabbitmq-plugins enable rabbitmq_management
    sudo deps/rabbit/scripts/rabbitmq-server 
    ```

  _**Note:** RabbitMQ management UI is available at `http://localhost:15672`._

#### References:
http://www.rabbitmq.com/
