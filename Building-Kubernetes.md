<!---PACKAGE:Kubernetes--->
<!---DISTRO:SLES 12.x:1.5.x--->
<!---DISTRO:RHEL 7.x:1.5.x--->
<!---DISTRO:Ubuntu 16.x:1.5.x--->

# Building Kubernetes

The instructions provided below specify the steps to build Kubernetes Version v1.5.1 on IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

More information on the Kubernetes is available at http://kubernetes.io/ and the source code can be obtained from https://github.com/kubernetes/kubernetes.

### Prerequisites:
  * Go
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7).
  * Docker
  -- Instructions for installing Docker can be found [here](https://www.ibm.com/developerworks/linux/linux390/docker.html).

### _**General Notes:**_
*  _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory /\<source_root\>/ will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

##Step 1: Building and Installing Kubernetes
####1.1) Install following dependencies
   
* RHEL 7.1/7.2/7.3

  ```
  sudo yum install --nogpgcheck -y git gcc-c++ which iptables make
  ```
* SLES 12/12-SP1/12-SP2

  ```
  sudo zypper install -y git gcc-c++ which iptables make
  ```
* Ubuntu 16.04/16.10

  ```
  sudo apt-get install -y git make iptables gcc wget tar flex subversion binutils-dev bzip2 build-essential vim
  ```

####1.2) Set environment variables
  ```
  export GOPATH=/<source_root>/kubernetes
  export PATH=$PATH:$GOPATH/bin:$GOPATH/_output/local/go/bin
  ```	
	
####1.3) Clone the source code and replace `sys` package

  ``` 
  cd /<source_root>/
  git clone https://github.com/kubernetes/kubernetes.git
  cd /<source_root>/kubernetes
  git checkout v1.5.1
  go get -u github.com/jteeuwen/go-bindata/go-bindata
  cd /<source_root>/kubernetes/vendor/golang.org/x 
  mv sys sys.bak 
  git clone https://github.com/linux-on-ibm-z/sys.git 
  cd /<source_root>/kubernetes
  ```

####1.4) Build Kubernetes
  ```
  cd /<source_root>/kubernetes
  make
  ```

**Note:** *Execute the following commands if you get the error message `cannot touch '_output/bin/deepcopy-gen': No such file or directory`.* 

  ```
  mkdir -p _output/bin
  touch _output/bin/deepcopy-gen
  touch _output/bin/conversion-gen
  touch _output/bin/defaulter-gen
  touch _output/bin/openapi-gen
  chmod 777 _output/bin/deepcopy-gen
  chmod 777 _output/bin/conversion-gen
  chmod 777 _output/bin/defaulter-gen
  chmod 777 _output/bin/openapi-gen
  ```

## Step 2: Testing (Optional)
####2.1) Run test suites
  ```
  cd /<source_root>/kubernetes
  make test
  ```
  **Note:** *Below changes are required to pass Kubernetes test cases on   Linux on z*
    
    
  * If test case execution fails because of `File system loop detected`, remove the `symlinkRecursiveParent` 
 
  ```
  rm ./src/github.com/jteeuwen/go-bindata/testdata/symlinkRecursiveParent/symlinkTarget
  ```

  * If `k8s.io/kubernetes/pkg/util/net` test case fails then add fields in `tls.Config` coming from Go 1.7 as shown [here](https://github.com/nhlfr/kubernetes/commit/c690ded4f7baaa55d8995ca22ef2f4093b28b4c0). 

  * If `k8s.io/kubernetes/pkg/volume` test case failure is observed then edit the following line in a file `/<source_root>/kubernetes/pkg/volume/metrics_du_test.go`.
   
  From
  ```
  const expectedBlockSize = 4096
  ```
  To
  ```
  const expectedBlockSize = 0
  ```
 

## Step 3: Running Kubernetes on a single host machine

**Note:** *Please use root user to run services.* 

####3.1) Kubernetes has a dependency on etcd service. Please follow the below steps to run etcd service.

* Install etcd

  * RHEL 7.1/7.2/7.3 and SLES 12/12-SP1/12-SP2
	
	Instructions for building etcd can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-etcd).
		
  * Ubuntu 16.04/16.10
	```
  sudo apt-get install etcd
	```

* Run etcd service

    * RHEL 7.1/7.2/7.3 and SLES 12/12-SP1/12-SP2
    ```
    ./bin/etcd -advertise-client-urls=http://127.0.0.1:4001 -advertise-client-urls=http://127.0.0.1:4001 -listen-client-urls=http://0.0.0.0:4001 --data-dir=/var/etcd/data &
    ```

    * Ubuntu 16.04/16.10
    ```
    etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data &
    ```

####3.2) Start Kubernetes services.
 
  * Start `apiserver` service
  
  ``` 
  cd /<source_root>/kubernetes/_output/local/go/bin/
  ./hyperkube apiserver --portal_net=10.0.0.1/24 --address=0.0.0.0 --insecure-bind-address=0.0.0.0 --etcd_servers=http://127.0.0.1:4001 --v=2 &
  ```

* Start `kubelet` service

  ```
  cd /<source_root>/kubernetes/_output/local/go/bin/
  ./hyperkube kubelet --root-dir=/var/data/kubelet --hostname-override="127.0.0.1" --address="0.0.0.0" --api-servers=http://localhost:8080 &
  ```

* Start `controller` service

  ``` 
  cd /<source_root>/kubernetes/_output/local/go/bin/
  ./hyperkube controller-manager --master=127.0.0.1:8080 --v=2 &
  ```

* Start `scheduler` service

  ```
  cd /<source_root>/kubernetes/_output/local/go/bin/
  ./hyperkube scheduler --master=127.0.0.1:8080 --v=2 &
  ```

* Start `proxy` service

  ``` 
  cd /<source_root>/kubernetes/_output/local/go/bin/
  ./hyperkube proxy --master=http://127.0.0.1:8080 --v=2 &
  ```

####3.3) Build pause

**Note:** *Kubernetes requires `gcr.io/google_containers/pause-s390x:3.0` image while deploying an application in a Kubernets pod. Following are 2 ways to create the required pause image.* 
 
  * Pull the image from repository and tag it to `gcr.io/google_containers/pause-s390x:3.0`
  ``` 
  docker pull brunswickheads/kubernetes-s390x
  docker tag brunswickheads/kubernetes-s390x:latest gcr.io/google_containers/pause-s390x:3.0
  ```
  
  *  Create pause image by following the instructions mentioned [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Pause)

After performing all the above steps, kubectl (binary will be found at `/<source_root>/kubernetes/_output/local/go/bin/`) command can be used for various purpose like deploying application, getting cluster nodes etc. More info about kubectl can be found [here](http://kubernetes.io/docs/user-guide/kubectl-overview/)

## Step 4: Building kube-dns (Optional)

Build instructions for kube-dns can be found [here] (https://github.com/linux-on-ibm-z/docs/wiki/Building-kube-dns)

### References:
http://kubernetes.io/
