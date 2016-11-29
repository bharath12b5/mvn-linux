# Building kube-dnsmasq 


The instructions provided below specify the steps to build kube-dnsmasq 1.3 on IBM z Systems for RHEL 7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified_

* _A directory /\<source_root\>/ will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it_

### Step 1: Install the dependencies
   
*   RHEL 7.2/7.3

        sudo yum install -y git gcc-c++ which iptables make

    Install docker by following instruction [here](https://www.ibm.com/developerworks/linux/linux390/docker.html)

*   SLES 12-SP1/12-SP2

        sudo zypper install -y git gcc-c++ which iptables make docker


*   Ubuntu 16.04/16.10

        sudo apt-get install -y git make iptables gcc wget tar flex subversion binutils-dev bzip2 build-essential vim golang-go docker


### Step 2: Download the source code

    cd /<source_root>/
    git clone https://github.com/linux-on-ibm-z/contrib.git
    cd contrib
    git checkout 0.7.0-s390x
    cd dnsmasq


### Step 3: Build multiarch/qemu-user-static:register image for s390x architecture 
   
 * Download the source code

        cd /<source_root>/  
        git clone -b v2.6.0 https://github.com/multiarch/qemu-user-static.git
        cd qemu-user-static/register

 * Edit the dockerfile `/<source_root>/qemu-user-static/register/Dockerfile` to change base image 

  ```diff
  @@ -1,4 +1,4 @@ 
  -   FROM busybox  
  +   FROM s390x/busybox  
      ENV QEMU_BIN_DIR=/usr/bin  
      ADD ./register.sh /register  
      ENTRYPOINT ["/register"] 
  ```

 *  Build the image
 
        docker build -t multiarch/qemu-user-static:register .

### Step 4: Build image gcr.io/google_containerskube-cross:v1.6.2-2 for s390x architecture 

 * Create ubuntu_golang image 
   
   Create Dockerfile with the following contents
    
	```
	FROM s390x/ubuntu:16.04
    RUN apt-get update
    RUN apt-get install -y golang
    ```
	
   Create golang image using the below command

    ```
    docker build -t  ubuntu_golang .
	```

 * Create gcr.io/google_containers/kube-cross:v1.6.2-2 image
  
   Create Dockerfile with the following contents

    ```
    FROM ubuntu_golang

    ENV GOARM 6
    ENV KUBE_DYNAMIC_CROSSPLATFORMS \
    s390x

    ENV KUBE_CROSSPLATFORMS \
    linux/386 \
    linux/arm linux/arm64 \
    linux/ppc64le \
    linux/s390x \
    darwin/amd64 darwin/386 \
    windows/amd64 windows/386

    # Pre-compile the standard go library when cross-compiling. This is much easier now when we have go1.5+
    RUN for platform in ${KUBE_CROSSPLATFORMS}; do GOOS=${platform%/*} GOARCH=${platform##*/} go install std; done

    # Install g++, then download and install protoc for generating protobuf output
    RUN apt-get update \
    && apt-get install -y g++ rsync apt-utils curl  file git unzip autoconf libtool automake g++ make git bzip2 curl unzip zlib1g-dev etcd \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
    RUN mkdir -p /usr/local/src/protobuf \
    && cd /usr/local/src/protobuf \
    && git clone https://github.com/google/protobuf.git \
    && cd protobuf && git checkout -b v3.0.0 \
    && ./autogen.sh && ./configure \
    && make install \
    && ldconfig \
    && cd .. \
    && rm -rf protobuf \
    && protoc --version

    # Use dynamic cgo linking for architectures other than amd64 for the server platforms
    # More info here: https://wiki.debian.org/CrossToolchains
    RUN echo "deb http://emdebian.org/tools/debian/ jessie main" > /etc/apt/sources.list.d/cgocrosscompiling.list \
    && curl -s http://emdebian.org/tools/debian/emdebian-toolchain-archive.key | apt-key add - \
    && for platform in ${KUBE_DYNAMIC_CROSSPLATFORMS}; do dpkg --add-architecture ${platform}; done \
    && apt-get update \
    && apt-get install -y build-essential \
    #  && for platform in ${KUBE_DYNAMIC_CROSSPLATFORMS}; do apt-get install -y crossbuild-essential-${platform}; done \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

    # work around 64MB tmpfs size in Docker 1.6
    ENV TMPDIR /tmp.k8s

    # Get the code coverage tool, godep, and go-bindata
    RUN  mkdir $TMPDIR \
    && export GOPATH=$TMPDIR \
    && chmod a+rwx $TMPDIR \
    && chmod o+t $TMPDIR \
    && chmod o+t $TMPDIR \
    && go get golang.org/x/tools/cmd/cover \
    golang.org/x/tools/cmd/goimports \
    github.com/tools/godep \
    github.com/jteeuwen/go-bindata/go-bindata 
    ```
	
   Create kube-cross image using the below command

    ```
    docker build -t gcr.io/google_containers/kube-cross:v1.6.2-2 .
    ```
	
### Step 5: Build kube-dnsmasq

    cd /<source_root>/contrib/dnsmasq
    make ARCH=s390x

### Step 6: Test the kube-dnsmasq image (Optional)

####Run kube-dnsmasq in docker container

*   Create a container of kube-dnsmasq image in daemon mode using command

        docker run --name <name> <image_name> -d

*   Following output should appear on the screen

        dnsmasq: started, version 2.75 cachesize 150
        dnsmasq: compile time options: IPv6 GNU-getopt no-DBus no-i18n no-IDN DHCP DHCPv6 no-Lua TFTP no-conntrack ipset   auth no-DNSSEC loop-detect inotify
        dnsmasq: reading /etc/resolv.conf
        dnsmasq: using nameserver 9.12.16.2#53
        dnsmasq: read /etc/hosts - 7 addresses

####Run kube-dnsmasq in Kubernetes pod


* Build Kubernetes using steps mentioned [here] (https://github.com/linux-on-ibm-z/docs/wiki/Building-Kubernetes) till step 4

* Follow steps mentioned [here] (https://github.com/linux-on-ibm-z/docs/wiki/Building-Kubernetes) from "Running Kubernetes on a single host machine" section to start the cluster
   
* Create a pod.json with following contents

        {
            "kind": "Pod",
            "apiVersion": "v1",
            "metadata": {
            "name": "simple0"
        },
             "spec": {
           "containers": [
             {
              "name": "dnsmasq",
              "image": "gcr.io/google_containers/kube-dnsmasq-s390x:1.3",
              "ports": [
               {
                "containerPort": 8080,
                "protocol": "TCP"
               }
              ]   
             }
            ]
           }
        }
 
  _**NOTE**: kubectl binary and pod.json should be in the same folder_
   
* And run the pod on your cluster using kubectl
        
        $ kubectl create -f pod.json
        pods/simple0
        $ kubectl get pods -o wide
        NAME     READY     STATUS    RESTARTS   AGE  NODE
        simple0   0/1       Running   0          3s   node

### References:
https://coreos.com/kubernetes/docs/latest/deploy-addons.html
