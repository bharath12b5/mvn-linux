# Building Docker Distribution 2.x

Docker Distribution can be built and tested on Linux on z Systems (RHEL 7.1 ans SLES 12) by following these instructions.

### Step 1 : Install the Dependencies
Following are the build dependencies for Distribution. 

* git-core (SLES12) or git (RHEL7)
* gccgo

**Dependencies Installation Notes:**   
*  Git can be installed on SLES12 using the below command.
     
            zypper install git-core

*  Git can be installed on RHEL7 using the below command.

            yum install git
            
 *  To install Gccgo, please refer to the [gccgo](https://github.com/linux-on-ibm-z/docs/wiki/Building-gccgo) recipe.

## Step 2 : Get the source (checkout v2.0.1 or v2.1.1 release)
            mkdir -p /go/src/github.com/docker
            cd /go/src/github.com/docker
            git clone https://github.com/docker/distribution.git
            cd distribution
            git checkout v2.1.1
            
## Step 3 : Set environment variables
* Set DISTRIBUTION_DIR environment variable. 

            export DISTRIBUTION_DIR=/go/src/github.com/docker/distribution

* Set GOPATH environment variable.

            export GOPATH=/go
            export GOPATH=$DISTRIBUTION_DIR/Godeps/_workspace:$GOPATH
            
## Step 4 : Edit Makefile
Edit the Makefile to disable ```GO_LDFLAGS``` variable (since its not supported by gccgo) set in the file. Comment out the below line in **Makefile**.

        GO_LDFLAGS=-ldflags "-X `go list ./version`.Version $(VERSION)"


Change it to

        #GO_LDFLAGS=-ldflags "-X `go list ./version`.Version $(VERSION)"
        
## Step 5 : Build the Binaries

Run the below command to build the distribution binaries.

            make PREFIX=/go clean binaries
            
## Step 6 : Run the Test Suite
Issue below command for running testcases.

            make PREFIX=/go test
            
## Step 7 : Start the registry 
Use the below command to start docker registry. The yml filename might be different depends on which version of distribution.

        /go/bin/registry cmd/registry/config.yml

**Note:**

* Docker registry fetches the configuration from cmd/registry/config.yml . 
* The filesystem location at which docker registry stores the images is by default set as ```/var/lib/registry``` in config.yml.

### References:
https://github.com/docker/distribution
