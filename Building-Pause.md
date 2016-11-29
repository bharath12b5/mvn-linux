# Building Pause

The instructions provided below specify the steps to build Pause Version 3.0 on IBM z Systems for RHEL 7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10


### Prerequisites
  * Docker 
  -- Instructions for installing or downloading Docker can be found [here](https://www.ibm.com/developerworks/linux/linux390/docker.html)

### _**General Note**_

* When following the steps below please use a standard permission user unless otherwise specified

* A directory /\<source_root\>/ will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it

### Step 1: Install the dependencies
   
* RHEL 7.2/7.3

        sudo yum install -y git gcc gcc-c++ which make

* SLES 12-SP1/12-SP2

        sudo zypper install -y git gcc gcc-c++ which make docker

* Ubuntu 16.04/16.10

        sudo apt-get install -y git make gcc docker

### Step 2: Download the source code for kubernetes  

    cd /<source_root>/
    git clone https://github.com/kubernetes/kubernetes.git
    cd /<source_root>/kubernetes
    git checkout v1.3.7

### Step 3: Build Pause 

    cd /<source_root>/kubernetes/build/pause 
    gcc -o pause-s390x pause.c

### Step 4: Create `gcr.io/google_containers/pause-s390x:3.0` image using below Dockerfile

   * Create Dockerfile with the following contents 

        #Dockerfile to create Pause image
        FROM s390x/ubuntu:16.04
        COPY pause-s390x /pause
        ENTRYPOINT ["/pause"]

   * Create Pause image using the below command

        docker build --tag gcr.io/google_containers/pause-s390x:3.0 . 
