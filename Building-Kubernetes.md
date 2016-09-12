<!---PACKAGE:Kubernetes--->
<!---DISTRO:SLES 12:1.3.5--->
<!---DISTRO:RHEL 7.1:1.3.5--->
<!---DISTRO:Ubuntu 16.x:1.3.5--->

# Building Kubernetes

The instructions provided below specify the steps to build Kubernetes Version v1.3.5 on Linux on the IBM z Systems for RHEL 7, SLES 12 and Ubuntu 16.04.

More information on the Kubernetes is available at http://kubernetes.io/ and the source code can be obtained from https://github.com/kubernetes/kubernetes.

### Prerequisites:
  * Go (For RHEL7 and SLES12)
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go).
  * Docker
  -- Instructions for installing Docker can be found [here](https://www.ibm.com/developerworks/linux/linux390/docker.html).

### _**General Note:**_
i)  _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory /\<source_root\>/ will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

### Building and Installing Kubernetes
1. Install following dependencies
   
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
        $ sudo apt-get install -y git make iptables gcc wget tar flex subversion binutils-dev bzip2 build-essential vim golang-go
    ```


2. Clone the source code and replace sys package
    ``` 
        $ cd /<source_root>/
        $ git clone https://github.com/kubernetes/kubernetes.git
        $ cd /<source_root>/kubernetes
        $ git checkout v1.3.5
        $ cd /<source_root>/kubernetes/vendor/golang.org/x 
        $ mv sys sys.bak 
        $ git clone https://github.com/linux-on-ibm-z/sys.git 
        $ cd /<source_root>/kubernetes
    ```
 
3. Build Kubernetes

    ```
        $ make
    ```

4. Run test suites (Optional)
    ```
        $ make test
    ```
    **Note:** *Below changes are required to pass Kubernetes test cases on Linux on z*

   i) If `k8s.io/kubernetes/pkg/kubelet` test case failure is observed then edit the file `/<source_root>/kubernetes/pkg/kubelet/kubelet_test.go` and follow the steps below.

   a) Add `goruntime "runtime"` to import section as follow
   ```
        import(
        ...
        goruntime "runtime"
        )
   ```
   b) Edit the following section 

   From  
   ```
        OperatingSystem:         "linux",
        Architecture:            "amd64",
   ```  
   to  
   ```
        OperatingSystem:         goruntime.GOOS,
        Architecture:            goruntime.GOARCH,
   ```  

   ii) If `k8s.io/kubernetes/pkg/volume` test case failure is observed on Ubuntu 16.04 then edit the following line in a file `/<source_root>/kubernetes/pkg/volume/metrics_du_test.go`.

   From  
   ```
        const expectedBlockSize = 4096
   ```  
   to  
   ```
        const expectedBlockSize = 0
   ``` 

### Running Kubernetes on a single host machine

**Note:** *Please use root user to run services.* 

**Step 1:** Kubernetes has a dependency on etcd service. Please follow the below steps to run etcd service.

i) Checkout the etcd source code from repository
   ```
        $ cd /<source_root>/
        $ git clone -b release-2.1 https://github.com/coreos/etcd
        $ cd etcd
   ```

ii) Build and run etcd service
   ```
        $ ./build
        $ ./bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data &
   ```

**Step 2:** Start Kubernetes services.
  

i) Start `apiserver` service
   ``` 
        $ cd /<source_root>/kubernetes/_output/local/go/bin/
        $ ./hyperkube apiserver --portal_net=10.0.0.1/24 --address=0.0.0.0 --insecure-bind-address=0.0.0.0 --etcd_servers=http://127.0.0.1:4001  --v=2 &
   ```

ii) Start `kubelet` service
   ```
        $ ./hyperkube kubelet --root-dir=/var/data/kubelet --hostname-override="127.0.0.1" --address="0.0.0.0" --api-servers=http://localhost:8080 &
   ```

iii) Start `controller` service
   ``` 
        $ ./hyperkube controller-manager --master=127.0.0.1:8080 --v=2 &
   ```

iv) Start `scheduler` service
   ```
        $ ./hyperkube scheduler --master=127.0.0.1:8080 --v=2 &
   ```

v) Start `proxy` service
   ``` 
        $ ./hyperkube proxy --master=http://127.0.0.1:8080 --v=2 &
   ```

**Step 3:** Kubernetes requires `gcr.io/google_containers/pause-s390x:3.0` image present in docker library. To get the pause image in docker library, follow the steps below.


   ```
        $ docker pull brunswickheads/kubernetes-s390x
        $ docker tag brunswickheads/kubernetes-s390x:latest gcr.io/google_containers/pause-s390x:3.0
        
   ```

After performing all the above steps, kubectl (binary will be found at `/<source_root>/kubernetes/_output/local/go/bin/`) command can be used for various purpose like deploying application, getting cluster nodes etc. More info about kubectl can be found [here](http://kubernetes.io/docs/user-guide/kubectl/kubectl/).
