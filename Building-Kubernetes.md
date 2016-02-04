# Building Kubernetes

Kubernetes can be built for Linux on z Systems running RHEL 7.1 and SLES12 by following these instructions. Version 1.1.0 has been successfully built & tested this way. More information on the Kubernetes is available at http://kubernetes.io/ and the source code can be obtained from https://github.com/kubernetes/kubernetes.

### Prerequisites:
  * Go
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go).
  * Docker
  -- Instructions for installing Docker can be found [here](https://www.ibm.com/developerworks/linux/linux390/docker.html).

### _**General Note:**_
i)  _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory /\<source_root\>/ will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

### Building and Installing Kubernetes
1. Install following dependencies.
   
   For RHEL7.1
    ```
        $ sudo yum install --nogpgcheck -y git gcc-c++ which iptables make
    ```
   For SLES12
    ```
        $ sudo zypper install -y git gcc-c++ which iptables make
    ```

2. Checkout the code from repository.
    ``` 
        $ cd /<source_root>/
        $ git clone -b release-1.1 https://github.com/linux-on-ibm-z/kubernetes
        $ cd kubernetes
    ```

3. Build and test(optional) Kubernetes.
    ```
        $ make
        $ make test
    ```

### Running Kubernetes on a single host machine

**Step 1:** To run Kubernetes, it needs etcd service running. Please follow the steps below to run etcd service.

i) Checkout the etcd source code from repository.
   ```
        $ cd /<source_root>/
        $ git clone -b release-2.1 https://github.com/coreos/etcd
        $ cd etcd
   ```

ii) Build and run etcd service.
   ```
        $ ./build
        $ ./bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data&
   ```

**Step 2:** To start Kubernetes follow the commands below.
  

i) Start api server service.
   ``` 
        $ cd /<source_root>/kubernetes/_output/local/go/bin/
        $ ./hyperkube apiserver --portal_net=10.0.0.1/24 --address=0.0.0.0 --insecure-bind-address=0.0.0.0 --etcd_servers=http://127.0.0.1:4001 --cluster_name=kubernetes --v=2&
   ```

ii) Start kubelet service.
   ```
        $ ./hyperkube kubelet --containerized --root-dir=/var/data/kubelet --hostname-override="127.0.0.1" --address="0.0.0.0" --api-servers=http://localhost:8080 --config=/etc/kubernetes/manifests&
   ```

iii) Start controller service.
   ``` 
        $ ./hyperkube controller-manager --master=127.0.0.1:8080 --v=2&
   ```

iv) Start scheduler service.
   ```
        $ ./hyperkube scheduler --master=127.0.0.1:8080 --v=2&
   ```

v) Start proxy service.
   ``` 
        $ ./hyperkube proxy --master=http://127.0.0.1:8080 --v=2&
   ```

**Step 3:** Kubernetes requires gcr.io/google_containers/pause:0.8.0 image present in docker library. To create pause image for z Systems follow the steps below.


i) Create a directory with name "pause" and change a working directory to this newly created directory.
   ```
        $ cd /<source_root>/
        $ mkdir pause
        $ cd pause
   ```
ii) Create a file name called Dockerfile. Copy and paste the following content into the file and save it. 

   ```console
      # Build pause image.
      FROM go_image

      # The author 
      MAINTAINER LoZ Open Source Ecosystem (https://www.ibm.com/developerworks/community/groups/community/lozopensource)

      # Set env variables for go
      ENV PATH=$HOME/go/bin:$PATH

      # Install dependencies
      RUN yum install -y git make which gcc-c++

      # Change the workdir to home
      WORKDIR /home

      # Clone the kubernetes from github
      RUN git clone -b release-1.1 https://github.com/linux-on-ibm-z/kubernetes

      # Set the work directory to pause 
      WORKDIR /home/kubernetes/build/pause

      # Build pause
      RUN go build --ldflags '-extldflags "-static" -s' pause.go

      # copy pause to root and bin directory
      RUN cp pause /
      RUN cp pause /bin/

      # Set the entry point for pause image.
      ENTRYPOINT ["/pause"]
   ```
iii) Create a base image i.e. go_image mentioned in the above docker file.

_*Note:*_  It is assumed that rhel7/sles12 docker image is present in the docker library. If not, follow this [link](http://containerz.blogspot.co.uk/2015/03/creating-base-images.html) to create rhel7/sles12 docker image.
   ```
        $ docker run -it --name=go_install rhel7:latest bash
                                 OR
        $ docker run -it --name=go_install sles12:latest bash
   ```

iv) After getting a bash shell of a running container, follow [Go recipe](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go) file to install Go in the container.

v) Save the above created container.
   ```
        $ docker commit go_install go_image
   ```
vi) Create a docker image from the above docker file.
   ```
        $ docker build --tag=gcr.io/google_containers/pause:0.8.0 .
   ```
vii) Once it successfully builds the image, verify the same image is listed in the docker library using following command.
   ```
        $ docker images
   ```	 

After performing all the above steps, kubectl(binary will be found at `/<source_root>/kubernetes/_output/local/go/bin/` ) command can be used for various purpose like deploying application, getting cluster nodes etc. More info about kubectl can be found [here](http://kubernetes.io/v1.0/docs/user-guide/kubectl/kubectl.html).
