<!---PACKAGE:Hyperledger--->
<!---DISTRO:SLES 12:0.6--->
<!---DISTRO:RHEL 7.2:0.6-->

# Building Hyperledger

Hyperledger has been successfully built on Linux on z Systems.  The following instructions can be used for RHEL 7.2 and SLES 12.


_**General Notes:**_
_When following the steps below please use a standard permission user unless otherwise specified._


## Prerequisites: ##

Hyperledger is written in `GO` and uses `Docker` to run chain code. It is required to build `Go` and install `Docker` first. Node.js (npm) is used for unit and behave test.

- Go -- Instructions for building Go can be found below.
- Docker -- Instructions for installing Docker can be found [here](https://www.ibm.com/developerworks/linux/linux390/docker.html).
- Node.js -- Instructions for installing Node.js can be found [here](https://developer.ibm.com/node/sdk/).


## Build and Install Golang  ##
To build  golang first to get a bootstrap that is a built package. 

      cd /tmp
      wget --quiet --no-check-certificate https://storage.googleapis.com/golang/go1.7.1.linux-s390x.tar.gz
      tar -xvf go1.7.1.linux-s390x.tar.gz

Then get golang 1.6 and an optimization branch, and start to build. Here "*golang_home*" is the golang home directory.

      cd <golang_home> 
      git clone http://github.com/linux-on-ibm-z/go.git go
      cd go/src
      git checkout release-branch.go1.6-p256
      export GOROOT_BOOTSTRAP=/tmp/go
      ./make.bash
      rm -rf /tmp/go1.7.1.linux-s390x.tar.gz /tmp/go
      export GOROOT=<golang_home>



## Install Rocksdb and dependencies ##
Rocksdb is an open source DB software used by Hyperledger.

1) Install dependent software of  `Rocksdb`

    sudo zypper install libbz2-devel
    sudo zypper install snappy-devel         # for SLES

or

    sudo yum install bzip2-devel.s390x
    sudo yum install snappy-devel.s390x      # for RHEL

2) Build and Install `Rocksdb`

    cd /tmp
    git clone https://github.com/facebook/rocksdb.git
    cd rocksdb
    git checkout v4.1
    sed -i -e "s/-march=native/-march=z196/" build_tools/build_detect_platform
    sed -i -e "s/-momit-leaf-frame-pointer/-DDUMBDUMMY/" Makefile

    PORTABLE=1 make shared_lib && INSTALL_PATH=/usr/local make install-shared && ldconfig
    rm -rf /tmp/rocksdb


3) Install pip, behave and docker-compose

    install --upgrade pip
    pip install behave nose docker-compose
    pip install -I flask==0.10.1 python-dateutil==2.2 pytz==2014.3 pyyaml==3.10 couchdb==1.0 flask-cors==2.0.1 requests==2.4.3

## Build Hyperledger and docker images ##
1) Download Hyperledger source code. A directory *op_wk* will be referred to working home directory of Hyperledger in these instructions, this is a  writable directory anywhere you'd like to place it..

    mkdir  <op_wk>   # op_wk is home directory of hyperledger
    cd <op_wk>
    export GOPATH=<op_wk>
    mkdir src
    cd $GOPATH/src
    mkdir -p github.com/hyperledger
    cd github.com/hyperledger
    git clone https://github.com/vpaprots/fabric   # clone and check out master branch
    cd fabric
    git checkout debian_s390x_v0.6                 # check out v0.6 branch for s390x


2) Setup environment variables


    export GOROOT=<golang_home>            # <golang_home> is home directory of the installed golang
    export PATH=<golang_home>/bin:$PATH
    export GOPATH=<op_wk>                  # hyperledger home directory
 

3) Build executable `peer` and `membersrvc`

    cd $GOPATH/src/github.com/hyperledger/fabric/peer
    go build  -v                        # build peer
    cd $GOPATH/src/github.com/hyperledger/fabric/membersrvc
    go build  -o  membersrvc            # security server



## Build docker Images ##
Hyperledger uses docker container  to hold  each chaincode, we need  to build  docker images that contain  `Golang` on IBM z Systems and `Rocksdb`.



1) Start docker daemon

*Note:* `docker` commands may need root permission


    sudo su
    docker daemon  -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock  &


2) Build Hyperledger docker images

    cd $GOPATH/src/github.com/hyperledger/fabric/
    make clean peer          # build base and hyperledger/fabric-peer images
    
    # may need root permission as it calls docker
 

Use `docker images` to confirm that a few base images and a image named **hyperledger-peer** are generated.

#### Unit Test ####

    cd $GOPATH/src/github.com/hyperledger/fabric
    export PATH=$GOPATH/bin:$PATH
    make unit-test           # run unit test

#### Running multiple peers ####
Follow the instruction in [`Setting Up a Network`](http://hyperledger-fabric.readthedocs.io/en/latest/Setup/Network-setup) section  to run multiple peers
