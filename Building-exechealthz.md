#Building exechealthz

The instructions provided below specify the steps to build exechealthz version 0.7.0 on IBM z Systems for RHEL 7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10

### Prerequisites
  * Go (RHEL 7.2/7.3 & SLES 12-SP1/12-SP2)
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7)
  
  * Docker (RHEL 7.2/7.3)
     -- Instructions for building docker can be found [here](https://www.ibm.com/developerworks/linux/linux390/docker.html)

_**General Notes:**_  
* _When following the steps below please use a standard permission user with sudo access unless otherwise specified_

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_

###  Step 1: Install the Dependencies

* RHEL 7.2/7.3

        sudo yum install wget gcc tar git make

* SLES 12-SP1/12-SP2

        sudo zypper install wget gcc tar git make docker

* Ubuntu 16.04/16.10

        sudo apt-get install wget gcc tar git make docker golang

### Step 2: Download contrib source code

        cd /<source_root>/
        git clone https://github.com/linux-on-ibm-z/contrib.git
        cd contrib
        git checkout 0.7.0-s390x

### Step 3: Test Health Server setup(Optional)

   _**Note:** Build golang image for s390x with below content(based on dockerfile from [here](https://github.com/docker-library/golang/blob/4fd5df86eea53623b1009b3621b40a97f9f359e5/1.7/Dockerfile))_

        FROM s390x/ubuntu:16.04

        # gcc for cgo
        RUN apt-get update && apt-get install -y --no-install-recommends \
                        wget \
                        g++ \
                        gcc \
                        libc6-dev \
                        make \
                        pkg-config \
                && rm -rf /var/lib/apt/lists/*

        ENV GOLANG_VERSION 1.7.3
        ENV GOLANG_DOWNLOAD_URL https://golang.org/dl/go$GOLANG_VERSION.linux-s390x.tar.gz

        RUN wget "$GOLANG_DOWNLOAD_URL" --no-check-certificate \
                && mv go$GOLANG_VERSION.linux-s390x.tar.gz golang.tar.gz \
                && tar -C /usr/local -xzf golang.tar.gz \
                && rm golang.tar.gz

        ENV GOPATH /go
        ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

        RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
        WORKDIR $GOPATH

        COPY go-wrapper /usr/local/bin/

  Command to build golang image

        docker build -t golang:1.7 .

_**Note:**_

  * go-wrapper can be found at https://github.com/docker-library/golang, please checkout go-wrapper in the same folder as dockerfile

  * Refer [Readme](https://github.com/linux-on-ibm-z/contrib/tree/0.7.0-s390x/exec-healthz) for creating images, please note that ARCH=s390x needs to be used in `make` command instead of ARCH=amd64 as used in existing [Readme](https://github.com/linux-on-ibm-z/contrib/tree/0.7.0-s390x/exec-healthz)

### References:
* http://kubernetes.io/
