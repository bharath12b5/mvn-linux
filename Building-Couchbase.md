<!---PACKAGE:Couchbase--->
<!---DISTRO:SLES 12.x:4.1.0--->
<!---DISTRO:RHEL 7.x:4.1.0--->
<!---DISTRO:Ubuntu 16.x:4.1.0--->

# Building Couchbase

Couchbase 4.1.0 can be built and tested on Linux on z Systems (RHEL 7.1/7.2/7.3, SLES 12/12-SP1 and Ubuntu 16.04/16.10) by following these instructions.

_**General Notes:**_  

* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##### Step 1 : Install the Dependencies


*	RHEL 7.1/7.2/7.3
     
		sudo yum install wget tar git gcc-c++ make curl openssl-devel libevent-devel libcurl-devel libicu-devel snappy-devel java-1.7.1-ibm java-1.7.1-ibm-devel ncurses-devel openssl unixODBC unixODBC-devel cmake

    
*	SLES 12/12-SP1

        sudo zypper install cmake wget tar git gcc-c++ make curl libopenssl-devel java-1.7.1-ibm-devel libevent-devel libcurl-devel libicu-devel snappy-devel ncurses-devel openssl unixODBC unixODBC-devel python-xml  

*   Ubuntu 16.04/16.10

		sudo apt-get install wget tar git g++ make curl libssl-dev libevent-dev libcurl4-openssl-dev libicu-dev libsnappy-dev ncurses-dev openssl libiodbc2-dev cmake


	Install java using the below link **(Ubuntu 16.04/16.10 Only)**
    
	Download the Java SDK 8 Installable package for Linux on z Systems 64-bit from [here](https://developer.ibm.com/javasdk/downloads/#tab_sdk8) and use the command below to install java  
		
		chmod +x ibm-java-s390x-sdk-8.0-3.20.bin
		sudo ./ibm-java-s390x-sdk-8.0-3.20.bin	
		export JAVA_HOME=<path_to_java_installation_directory>
		export PATH=$JAVA_HOME/bin:$PATH

		
**Other Dependencies**

*	To install Go, please refer to the [Go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7) recipe

*   To install Erlang, please refer to 
  
    [Erlang RHEL7](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang-on-RHEL7)  recipe 
	
	[Erlang SLES12](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang-on-SLES12) recipe     
	
	For Ubuntu 16.04/16.10 follow the instructions below
	
		cd /<source_root>/
	
	As Root user,
	
		wget http://www.erlang.org/download/otp_src_17.4.tar.gz  
	
	As regular user,
        	
		tar zxvf otp_src_17.4.tar.gz
		cd /<source_root>/otp_src_17.4
		export ERL_TOP=`pwd`
		./configure --prefix=/usr
		make
		
	As Root user,
		
		cd /<source_root>/otp_src_17.4
		make install
		
    
*   To install V8 libraries, please refer to the [V8 3.14](https://github.com/linux-on-ibm-z/docs/wiki/Building-V8-libraries) recipe

##### Step 2 : Update configuration to use gcc-6 and g++-6 **(Ubuntu 16.10 Only)**    
     
	  sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 40
	  sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 40
	  sudo update-alternatives --config gcc
	  sudo update-alternatives --config g++
	  export PATH=$PATH:/usr/local/go/bin
		
##### Step 3 : Download the Repo tool

*   Download the Repo tool using cURL tool and ensure that it has execute permissions
        
		cd /<source_root>/
		curl https://storage.googleapis.com/git-repo-downloads/repo > repo
		chmod a+x repo
		
            
##### Step 4 : Build, install and test Couchbase

*	Create a directory couchbase

		mkdir -p /<source_root>/couchbase

		
*	Clone the Couchbase 4.1.0 release using the the manifest file via Repo tool with the init and sync commands
		
		cd /<source_root>/couchbase
		git config --global user.email your@email.addr
		git config --global user.name  your_name
		../repo init -u git://github.com/couchbase/manifest -m released/4.1.0.xml
        ../repo sync

		
*   Edit the following files
    
1. `/<source_root>/couchbase/tlm/cmake/Modules/CouchbaseDefinitions.cmake`
	
    	Add the following line in the file
	
			ADD_DEFINITIONS(-DWORDS_BIGENDIAN=1)
	
2. `/<source_root>/couchbase/forestdb/src/arch.h`
	
		In section  `#elif __linux__`  add following lines		
        
	         #if defined(__s390x__)
             	#undef SPIN_INITIALIZER
             	#define SPIN_INITIALIZER (spin_t)(0)
             #endif
			 
		after line
		
			#define SPIN_INITIALIZER (spin_t)(1)
			             
3. `/<source_root>/couchbase/couchstore/src/views/bin/couch_view_file_merger.c`
    
    	Replace line ``}merge_file_type_t;``  with  ``};typedef unsigned char merge_file_type_t;``
	
 
*   Run the make command and start the building procedure

		cd /<source_root>/couchbase/
		make
		
		
*   Run the test cases using ctest tool

		cd /<source_root>/couchbase/build/
		ctest
            

##### Step 4 : Start the server
			
*	Use the below command to start Couchbase Server

		/<source_root>/couchbase/install/bin/couchbase-server -- -noinput &
		
		
	We can view the  Couchbase UI at   http://localhost:8092


##### References:
http://www.couchbase.com/nosql-databases/couchbase-server
