# Building Go  

Below versions of Go are available in respective distributions at the time of this recipe creation:   

* Ubuntu 16.10 has `1.7` 
* Ubuntu 16.04 has `1.6`

The instructions provided below specify the steps to build Go version 1.7.3 on IBM z Systems for 
RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_   
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._


##Step 1: Building and Installing Go 
####1.1) Install dependencies  

   * RHEL 6.8/7.1/7.2/7.3  

		sudo yum install tar wget 

   * SLES 11-SP4/12/12-SP1/12-SP2  

		sudo zypper install tar wget
   
   * Ubuntu 16.04/16.10  

		sudo apt-get update
		sudo apt-get install wget tar

####1.2) Download source code

	cd <source_root>
	wget https://storage.googleapis.com/golang/go1.7.3.linux-s390x.tar.gz
	tar -xzf go1.7.3.linux-s390x.tar.gz

####1.3) Set environment variable

	export PATH=$PATH:/<source_root>/go/bin
	export GOROOT=/<source_root>/go
	export CC=gcc


## Step 2: Testing (Optional)
####2.1)  Command `go version` should output

    go version go1.7.3 linux/s390x

### References:
https://golang.org/
