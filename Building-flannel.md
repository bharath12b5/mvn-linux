<!---PACKAGE:flannel--->
<!---DISTRO:SLES 12.x:0.5.5--->
<!---DISTRO:RHEL 7.x:0.5.5--->
<!---DISTRO:Ubuntu 16.x:0.5.5--->


# Building flannel


The instructions provided below specify the steps to build flannel 0.5.5 on IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10

### Prerequisites
  * Go (RHEL 7.1/7.2/7.3 & SLES 12/12-SP1/12-SP2)
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7)

_**General Notes:**_  
_When following the steps below please use a standard permission user unless otherwise specified._



#### Step 1: Install the Dependencies

* RHEL 7.1/7.2/7.3

        sudo yum install -y git make wget gcc   
   
* SLES 12/12-SP1/12-SP2

        sudo zypper install -y git make wget gcc

* Ubuntu 16.04/16.10
		
        sudo apt-get install -y golang git make gcc
		
#### Step 2: Build and install flannel
* Get the source
    ```bash
        export GOPATH=$HOME/gopath
        export PATH=$HOME/gopath/bin:$PATH
        mkdir -p $HOME/gopath/src/github.com/coreos
        cd $HOME/gopath/src/github.com/coreos/
        git clone https://github.com/linux-on-ibm-z/flannel
        cd $HOME/gopath/src/github.com/coreos/flannel
        git checkout v0.5.5-s390x
    ```

* Build flannel
    ```bash
        cd $HOME/gopath/src/github.com/coreos/flannel
        ./build
    ```
* Run test cases(Optional)
    ```bash
        cd $HOME/gopath/src/github.com/coreos/flannel
        ./test
    ```

   _**Note**: Test case `github.com/coreos/flannel/remote` failure is not related to IBM z Systems_
