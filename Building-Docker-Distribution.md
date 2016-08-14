<!---PACKAGE:Docker Distribution--->
<!---DISTRO:SLES 12:2.5.0--->
<!---DISTRO:RHEL 7.1:2.5.0--->
<!---DISTRO:Ubuntu 16.x:2.5.0--->

# Building Docker Distribution
Below versions of Docker Distribution are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `2.3.0`

The instructions provided below specify the steps to build Docker Distribution 2.5.0 on Linux on the IBM z Systems for RHEL 7.1, SLES 12 and Ubuntu 16.04.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##### Step 1 : Install the Dependencies
Following are the build dependencies for Distribution. 

* git-core (SLES 12) or git (RHEL 7.1, Ubuntu 16.04)
* Go
* make

**Dependencies Installation Notes:**   
*	SLES 12
     
            sudo zypper install -y git-core make

*	RHEL 7.1

            sudo yum install -y git make

*	Ubuntu 16.04

            sudo apt-get install -y git make golang
            
*	For RHEL7, SLES12: To install Go, please refer to the [Go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go) recipe.

##### Step 2 : Get the source (checkout v2.5.0 release)
*	Create a distribution directory and clone the source code there.
			
			mkdir -p /<source_root>/src/github.com/docker
			cd /<source_root>/src/github.com/docker
			git clone https://github.com/docker/distribution.git
			cd /<source_root>/src/github.com/docker/distribution
			git checkout v2.5.0
            
##### Step 3 : Set environment variable
*	Set DISTRIBUTION_DIR environment variable. 

			export DISTRIBUTION_DIR=/<source_root>/src/github.com/docker/distribution

*	Set GOPATH environment variable.

			export GOPATH=/<source_root>/
            
##### Step 4 : Build the Binaries
*	Run the below command to build the distribution binaries.

			make PREFIX=/<source_root>/ clean binaries
            
##### Step 5 : Run the Test Suite
*	Issue below command for running testcases.

            make PREFIX=/<source_root>/ test
            
##### Step 6 : Start the registry 

Docker registry fetches the configuration from cmd/registry/config.yml. 
The filesystem location at which docker registry stores the images is by default set as ```/var/lib/registry``` in config.yml.
*	Copy the config-dev.yml file to config.yml

			cp $DISTRIBUTION_DIR/cmd/registry/config-dev.yml $DISTRIBUTION_DIR/cmd/registry/config.yml

* 	Create directory to store images (if it does not exist)

			sudo mkdir -p /var/lib/registry
			sudo chown <user> /var/lib/registry
			
*	Use the below command to start docker registry

			/<source_root>/bin/registry serve $DISTRIBUTION_DIR/cmd/registry/config.yml



##### References:
https://github.com/docker/distribution