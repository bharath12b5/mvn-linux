# Building flannel


The instructions provided below specify the steps to build flannel 0.5.5 on IBM z Systems for RHEL 7.2, SLES 12-SP1 and Ubuntu 16.04/16.10

### Prerequisites
  * Go (RHEL 7.2 & SLES 12-SP1)
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7)

_**General Notes:**_  
_When following the steps below please use a standard permission user unless otherwise specified._


#### Step 1: Install the Dependencies

* RHEL 7.2

        sudo yum install -y git make wget gcc   
   
* SLES 12-SP1

        sudo zypper install -y git make wget gcc

* Ubuntu 16.04/16.10
		
        sudo apt-get install -y golang git make gcc
		
#### Step 2: Build and install flannel
* Get the source

        export GOPATH=$HOME/gopath
        export PATH=$HOME/gopath/bin:$PATH
        mkdir -p $HOME/gopath/src/github.com/coreos
        cd $HOME/gopath/src/github.com/coreos/
        git clone https://github.com/linux-on-ibm-z/flannel
        cd $HOME/gopath/src/github.com/coreos/flannel
        git checkout v0.5.5-s390x
 

* Build flannel

        cd $HOME/gopath/src/github.com/coreos/flannel
        ./build

* Run test cases(Optional)
           
        cd $HOME/gopath/src/github.com/coreos/flannel
        ./test


   _**Note**: There is one test case(Module `github.com/coreos/flannel/remote`) failures which is also observed on Intel x86. This failure can be ignored._
