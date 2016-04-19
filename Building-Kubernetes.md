<!---PACKAGE:Kubernetes--->
<!---DISTRO:SLES 12:1.1.0--->
<!---DISTRO:RHEL 7.1:1.1.0--->
<!---DISTRO:Ubuntu 16.x:1.1.0--->

# Building Kubernetes

Kubernetes can be built for Linux on z Systems running on RHEL7, SLES12 and Ubuntu 16.04 by following these instructions. Version 1.1.0 has been successfully built & tested this way. More information on the Kubernetes is available at http://kubernetes.io/ and the source code can be obtained from https://github.com/kubernetes/kubernetes.

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
   
   For RHEL7
    ```
        $ sudo yum install --nogpgcheck -y git gcc-c++ which iptables make
    ```
   For SLES12
    ```
        $ sudo zypper install -y git gcc-c++ which iptables make
    ```
   For Ubuntu 16.04
    ```
        $ sudo apt-get install -y git make iptables gcc wget tar flex subversion binutils-dev bzip2 build-essential golang-go
    ```


2. Checkout the code from repository.
    ``` 
        $ cd /<source_root>/
        $ git clone -b release-1.1 https://github.com/linux-on-ibm-z/kubernetes
        $ cd kubernetes
    ```
    **Note:** *Below changes required only on Ubuntu 16.04 to install kubernetes on Linux on z.*

    i) Modify below changes in  `/<source_root>/kubernetes/Godeps/_workspace/src/github.com/docker/libcontainer/netlink/netlink_linux_notarm.go` file.
 
    From  
    ```
    // +build !arm,!s390x,!ppc64,!ppc64le
    ```  

    to  

    ```
    // +build !arm,!ppc64,!ppc64le
    ```  

    ii) `rm /<source_root>/kubernetes/Godeps/_workspace/src/github.com/docker/libcontainer/netlink/netlink_linux_arms390x.go` 


3. Build Kubernetes.

    ```
        $ make
    ```

4. Run test suites (Optional).
    ```
        $ make test
    ```

### Running Kubernetes on a single host machine

**Note:** *Please use a root permission user to run services.* 

**Step 1:** Kubernetes have a dependency on etcd service. Please follow below steps to run etcd service.

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

**Step 2:** Start Kubernetes services.
  

i) Start `apiserver` service.
   ``` 
        $ cd /<source_root>/kubernetes/_output/local/go/bin/
        $ ./hyperkube apiserver --portal_net=10.0.0.1/24 --address=0.0.0.0 --insecure-bind-address=0.0.0.0 --etcd_servers=http://127.0.0.1:4001 --cluster_name=kubernetes --v=2&
   ```

ii) Start `kubelet` service.
   ```
        $ ./hyperkube kubelet --containerized --root-dir=/var/data/kubelet --hostname-override="127.0.0.1" --address="0.0.0.0" --api-servers=http://localhost:8080 --config=/etc/kubernetes/manifests&
   ```

iii) Start `controller` service.
   ``` 
        $ ./hyperkube controller-manager --master=127.0.0.1:8080 --v=2&
   ```

iv) Start `scheduler` service.
   ```
        $ ./hyperkube scheduler --master=127.0.0.1:8080 --v=2&
   ```

v) Start `proxy` service.
   ``` 
        $ ./hyperkube proxy --master=http://127.0.0.1:8080 --v=2&
   ```

**Step 3:** Kubernetes requires gcr.io/google_containers/pause:0.8.0 image present in docker library. To create pause image for Linux on z follow the steps below.


   ```
        $ cd /<source_root>/kubernetes/build/pause
		$ sed --in-place '/go get github.com/,/goupx/d' prepare.sh
        $ make build
   ```

After performing all the above steps, kubectl (binary will be found at `/<source_root>/kubernetes/_output/local/go/bin/`) command can be used for various purpose like deploying application, getting cluster nodes etc. More info about kubectl can be found [here](http://kubernetes.io/v1.0/docs/user-guide/kubectl/kubectl.html).