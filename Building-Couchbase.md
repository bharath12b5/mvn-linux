<!---PACKAGE:Couchbase--->
<!---DISTRO:SLES 12:4.1.0--->
<!---DISTRO:RHEL 7.1:4.1.0--->


# Building Couchbase

Couchbase 4.1.0 can be built and tested on Linux on z Systems (RHEL 7.1 and SLES 12) by following these instructions.

_**General Notes:**_  

i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##### Step 1 : Install the Dependencies


*	RHEL7
     
		sudo yum install wget tar git gcc-c++ make curl openssl-devel libevent-devel libcurl-devel libicu-devel snappy-devel java-1.7.1-ibm java-1.7.1-ibm-devel ncurses-devel openssl unixODBC unixODBC-devel cmake

    
*	SLES12

        sudo zypper install cmake wget tar git gcc-c++ make curl libopenssl-devel java-1.7.1-ibm-devel java-1.7.1-ibm-devel libevent-devel libcurl-devel libicu-devel snappy-devel ncurses-devel openssl unixODBC unixODBC-devel python-xml  

		
**Other Dependencies**

*	To install Go, please refer to the [Go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go) recipe

*   To install Erlang, please refer to 
  
    [Erlang RHEL7](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang-on-RHEL7)  recipe 
	
	[Erlang SLES12](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang-on-SLES12) recipe     
	
    
*   To install V8 libraries, please refer to the [V8 3.14](https://github.com/linux-on-ibm-z/docs/wiki/Building-V8-libraries) recipe

		
##### Step 2 : Download the Repo tool

*   Download the Repo tool using cURL tool and ensure that it has execute permissions
        
		cd /<source_root>/
		curl https://storage.googleapis.com/git-repo-downloads/repo > repo
		chmod a+x repo
		
            
##### Step 3 : Build, install and test Couchbase

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
	
	2. `/<source_root>/couchbase/tlm/cmake/Modules/FindCouchbaseGo.cmake`
	
    	Replace the line 
	
			MESSAGE(FATAL_ERROR "Go version of ${GO_MINIMUM_VERSION} or higher required (found version ${GO_VERSION})")
		
		With following lines
	
            STRING(REGEX MATCH "^go version devel .*" go_dev_version "${GO_VERSION}")
            IF (go_dev_version)
              MESSAGE(STATUS "WARNING: You are using a development version of go")
              MESSAGE(STATUS "         Go version of ${GO_MINIMUM_VERSION} or higher required")
              MESSAGE(STATUS "         You may experience problems caused by this")
            ELSE(go_dev_version)
              MESSAGE(FATAL_ERROR "Go version of ${GO_MINIMUM_VERSION} or higher required (found version ${GO_VERSION})")
            ENDIF(go_dev_version)
		     
	3. `/<source_root>/couchbase/forestdb/src/arch.h`
	
		In section  `#elif __linux__`  add following lines		
        
	         #if defined(__s390x__)
             	#undef SPIN_INITIALIZER
             	#define SPIN_INITIALIZER (spin_t)(0)
             #endif
			 
		after line
		
			#define SPIN_INITIALIZER (spin_t)(1)
			             
    4. `/<source_root>/couchbase/couchstore/src/views/bin/couch_view_file_merger.c`
    
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