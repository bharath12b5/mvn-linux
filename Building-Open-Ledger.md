<!---PACKAGE:Open Ledger--->
<!---DISTRO:SLES 12:0.1.0--->
<!---DISTRO:RHEL 7.1:0.1.0--->

# Building Open Ledger

Open Ledger version 0.1.0 has been successfully built on Linux on z Systems.  The following instructions can be used for RHEL 7 and SLES 12.

_**General Notes:**_
_When following the steps below please use a standard permission user unless otherwise specified._

## Prerequisites: ##

Open ledger is written in `GO` and uses `Docker` to run each peer. It is required to build `Go` and install `Docker` first.

- Go -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go).
- Docker -- Instructions for installing Docker can be found [here](https://www.ibm.com/developerworks/linux/linux390/docker.html).
- Docker Distribution -- Instructions for building and starting Docker Distribution  can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Docker-Distribution).


## Build and Install Rocksdb  ##
Rocksdb is an open source DB software used by Open Ledger.

1) Download and build Rocksdb code

    git clone https://github.com/facebook/rocksdb.git
    cd  rocksdb
    git checkout v4.1
    sed -i -e "s/-march=native/-march=z196/" build_tools/build_detect_platform
    sed -i -e "s/-momit-leaf-frame-pointer/-DDUMMY/" Makefile
    make shared_lib
2) Copy rocksdb to  path /opt

    sudo cp -r rocksdb /opt/


## Build Open Ledger ##
1) Download Open ledger code. *A directory <op_wk\> will be referred to working home directory of Open Ledger in these instructions, this is a temporary writable directory anywhere you'd like to place it..*

    mkdir  <op_wk>   # op_wk is home directory of open ledger
    cd <op_wk>
    export GOPATH=<op_wk>
    mkdir src
    cd $GOPATH/src
    mkdir -p github.com/openblockchain
    cd github.com/openblockchain
    git clone https://github.com/openblockchain/obc-peer
    cd obc-peer
    git checkout tags/v0.1.0       # checkout to v0.1.0 code branch

2) Setup environment variables
  

    export GOROOT=<golang_home>           # <golang_home> is home directory of the installed golang
    export GO15VENDOREXPERIMENT=1
    export PATH=<golang_home>/bin:$PATH
    export GOPATH=<op_wk>                  # open ledger home directory
    export CGO_LDFLAGS="-L/opt/rocksdb -lrocksdb -lstdc++ -lm -lz "
    export CGO_CFLAGS="-I/opt/rocksdb/include"
    export LD_LIBRARY_PATH=/opt/rocksdb:$GOPATH/rocksdb:$LD_LIBRARY_PATH

3) Build executable `obc-peer`, `chaincode_example02` and `obcca-server`

    cd $GOPATH/src/github.com/openblockchain/obc-peer
    go build  -v
    go build  -v   openchain/example/chaincode/chaincode_example02/chaincode_example02.go
    cd $GOPATH/src/github.com/openblockchain/obc-peer/obc-ca
    go build -o obcca-server   # security server


It is possible to run one peer in the development environment by following the instruction   in [Sandbox Setup ](https://github.com/openblockchain/obc-docs/blob/master/api/SandboxSetup.md) from the section **Security Setup (optional)**.

*Note:* Each term (terminal) has to setup environment variables as shown above and is able to ignore `vagrant`


## Build Golang Docker Image ##
Open ledger uses docker container  to hold  each peer of the Open ledger, we need  to build a docker image of Golang on IBM z Systems

1)  Build and start Docker Distribution on a host

Follow the [Docker Distribution](https://github.com/linux-on-ibm-z/docs/wiki/Building-Docker-Distribution) instructions to build and start docker registry. 

Find the host IP address and keep it as  `<docker_registry_host_ip>`

Start the registry with the command below:

    /<source_root>/bin/registry /<source_root>/src/github.com/docker/distribution/cmd/registry/config.yml

*Note:* Docker Distribution uses the default port number `5000`, make sure it is not blocked by firewall so that this host can be accessed via this port.    

2) Log in to another host,  create a docker file <docker_file\> and its content is as follows:

    FROM s390x/debian
    # gcc for cgo
    RUN apt-get update && apt-get install -y --no-install-recommends \
                g++ \
                gcc \
                libc6-dev \
                make \
                curl \
        && rm -rf /var/lib/apt/lists/*
    # assume `./go` is golang home direcotry
    COPY go /usr/local/go
    ENV GOPATH /go
    ENV GOROOT /usr/local/go
    ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

    RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
    WORKDIR $GOPATH

3) Start docker daemon  and build golang image

*Note:* `docker` commands may need root permission


    sudo su
    docker daemon --dns <DNS_IP> -s=aufs -r=true --api-enable-cors=true -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock --insecure-registry <docker_registry_host_ip>:5000 &

    cd wk1       # work directory
    cp -r <golang-home> go    # make golang directory as "./go"
    docker build -t <docker_registry_host_ip>:5000/s390x/golang -f <docker_file> .
An new image named as  `<docker_registry_host_ip>:5000/s390x/golang` is created. ( using command `docker images` to confirm)


4)  Push the image to the docker registry

The generated docker golang image is required to push to a private docker distribution, which IP address is `<docker_registry_host_ip>`


    docker push  <docker_registry_host_ip>:5000/s390x/golang

Now the new golang docker image is available for building Open Ledger docker image

## Build an Open Ledger docker image ##

1) Go back to the obc_peer directory

    cd $GOPATH/src/github.com/openblockchain/obc-peer

2) Update `openchain.yaml`, in section `peer:` change the `Dockerfile:` part as follows:

    Dockerfile:  |
        from <docker_registry_host_ip>:5000/s390x/golang
        ENV GO15VENDOREXPERIMENT=1
        # Install RocksDB
        RUN apt-get update && apt-get install -y git
        RUN cd /opt  && git clone --branch v4.1 --single-branch --depth 1 https://github.com/facebook/rocksdb.git && cd rocksdb
        WORKDIR /opt/rocksdb
        RUN sed -i -e "s/-march=native/-march=z196/" build_tools/build_detect_platform
        RUN sed -i -e "s/-momit-leaf-frame-pointer/-DDUMMY/" Makefile
        RUN make shared_lib
        ENV LD_LIBRARY_PATH=/opt/rocksdb:$LD_LIBRARY_PATH
        RUN apt-get update && apt-get install -y libsnappy-dev zlib1g-dev libbz2-dev
        # Copy GOPATH src and install Peer
        COPY src $GOPATH/src
        RUN mkdir -p /var/openchain/db
        WORKDIR $GOPATH/src/github.com/openblockchain/obc-peer/
        RUN CGO_CFLAGS="-I/opt/rocksdb/include" CGO_LDFLAGS="-L/opt/rocksdb -lrocksdb -lstdc++ -lm -lz -lbz2 -lsnappy" go install && cp $GOPATH/src/github.com/openblockchain/obc-peer/openchain.yaml $GOPATH/bin

3) In section `chaincode:` change `Dockerfile:` part as

        Dockerfile:  |
            from <docker_registry_host_ip>:5000/s390x/golang
            ENV GO15VENDOREXPERIMENT=1
            COPY src $GOPATH/src
            WORKDIR $GOPATH

4) Build Open Ledger docker image

    cd $GOPATH/src/github.com/openblockchain/obc-peer/openchain/container
    go test -timeout=20m -run BuildImage_Peer          # may need root permission as it calls docker

A new image named **openchain-peer** is generated.

#### Unit Test ####

1) To run unit test, repeat updating `openchain.yaml` change above for these 2 files:

- `$GOPATH`/src/github.com/openblockchain/obc-peer/obc-ca/obcca/obcca_test.yaml

- `$GOPATH`/src/github.com/openblockchain/obc-peer/openchain/ledger/genesis/genesis_test.yaml


2) Update Line 96 of the file `$GOPATH`/src/github.com/openblockchain/obc-peer/openchain/container/controller_test.go, changing `busybox` to `s390x/busybox`.

Here is a sample diff patch describing the change:

    @@ -93,7 +93,7 @@ func getCodeChainBytesInMem() (io.Reader, error) {
             tr := tar.NewWriter(gw)
    -       dockerFileContents := []byte("FROM busybox:latest\n\nCMD echo hello")
    +       dockerFileContents := []byte("FROM s390x/busybox:latest\n\nCMD echo hello")
        dockerFileSize := int64(len([]byte(dockerFileContents)))

3) Follow the instructions in section [Unit Tests](https://github.com/openblockchain/obc-peer) to run the unit test.


#### Running multiple peers ####
Follow the instruction in [Setting Up a Network For Development](https://github.com/openblockchain/obc-docs/blob/master/dev-setup/devnet-setup.md) from section **Starting up validating peers** to run multiple peers 