# Building Docker Distribution

Docker Distribution 2.2.1 can be built and tested on Linux on z Systems (RHEL 7.1 and SLES 12) by following these instructions.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##### Step 1 : Install the Dependencies
Following are the build dependencies for Distribution. 

* git-core (SLES12) or git (RHEL7)
* Go

**Dependencies Installation Notes:**   
*	Git can be installed on SLES12 using the below command.
     
            sudo zypper install -y git-core

*	Git can be installed on RHEL7 using the below command.

            sudo yum install -y git
            
*	To install Go, please refer to the [Go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go) recipe.

##### Step 2 : Get the source (checkout v2.2.1 release)
*	Create a distribution directory and clone the source code there.
			
			mkdir -p /<source_root>/src/github.com/docker
			cd /<source_root>/src/github.com/docker
			git clone https://github.com/docker/distribution.git
			cd /<source_root>/src/github.com/docker/distribution
			git checkout v2.2.1
            
##### Step 3 : Set environment variable
*	Set DISTRIBUTION_DIR environment variable. 

			export DISTRIBUTION_DIR=/<source_root>/src/github.com/docker/distribution

*	Set GOPATH environment variable.

			export GOPATH=/<source_root>/
			export GOPATH=$DISTRIBUTION_DIR/Godeps/_workspace:$GOPATH
            
##### Step 4 : Build the Binaries
*	Run the below command to build the distribution binaries.

			make PREFIX=/<source_root>/ clean binaries
            
##### Step 5 : Run the Test Suite
*	Issue below command for running testcases.

            make PREFIX=/<source_root>/ test
            
##### Step 6 : Start the registry 
*	Copy the config-dev.yml file to config.yml

			cp $DISTRIBUTION_DIR/cmd/registry/config-dev.yml $DISTRIBUTION_DIR/cmd/registry/config.yml
			
*	Use the below command to start docker registry.

			/<source_root>/bin/registry $DISTRIBUTION_DIR/cmd/registry/config.yml

**Note:**

* Docker registry fetches the configuration from cmd/registry/config.yml . 
* The filesystem location at which docker registry stores the images is by default set as ```/var/lib/registry``` in config.yml.

##### References:
https://github.com/docker/distribution