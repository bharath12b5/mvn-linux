# Building kube-dns 


The instructions provided below specify the steps to build kube-dns 1.6 on IBM z Systems for RHEL 7.2/7.3,SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10

### Prerequisites:
  * Go (For RHEL 7.2/7.3 and SLES 12-SP1/12-SP2)
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7)

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified_

* _A directory /\<source_root\>/ will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it_

###  Step 1: Install the dependencies
   
   * RHEL 7.2/7.3

        sudo yum install -y git gcc-c++ which iptables make

    Install docker by following instruction [here](https://www.ibm.com/developerworks/linux/linux390/docker.html)

   * SLES 12-SP1/12-SP2

        sudo zypper install -y git gcc-c++ which iptables make docker

   * Ubuntu 16.04/16.10

        sudo apt-get install -y git make iptables gcc wget tar flex subversion binutils-dev bzip2 build-essential vim golang-go docker

###  Step 2: Set environment variables

	export GOPATH=/<source_root>/kubernetes
    export PATH=$PATH:$GOPATH/bin:$GOPATH/_output/local/go/bin

###  Step 3: Download the source code and replace sys package

	cd /<source_root>/
	git clone https://github.com/linux-on-ibm-z/kubernetes.git
    cd kubernetes
    git checkout v1.4.0-s390x
    go get -u github.com/jteeuwen/go-bindata/go-bindata
    cd vendor/golang.org/x 
    mv sys sys.bak 
    git clone https://github.com/linux-on-ibm-z/sys.git         

###  Step 4: Edit Makefile in Kubernetes `/<source_root>/kubernetes/Makefile`
 

  ```diff

@@ -39,7 +39,7 @@ MAKEFLAGS += --no-builtin-rules
#   Constants used throughout.
    .EXPORT_ALL_VARIABLES:
    OUT_DIR ?= _output
-   BIN_DIR := $(OUT_DIR)/bin
+   BIN_DIR := $(OUT_DIR)/local/go/bin
    PRJ_SRC_PATH := k8s.io/kubernetes
    GENERATED_FILE_PREFIX := zz_generated.
 
  ```
   
###  Step 5: Build Kubernetes

    cd /<source_root>/kubernetes
	make ARCH=s390x

###  Step 6: Edit `/<source_root>/kubernetes/build/kube-dns/Makefile`

```diff
@@ -49,7 +49,7 @@ all: container
 
 container:
 	# Copy the content in this dir to the temp dir
-	cp $(KUBE_ROOT)/_output/dockerized/bin/$(PLATFORM)/$(ARCH)/kube-dns $(TEMP_DIR)
+	cp /<source_root>/kubernetes/_output/local/go/bin/kube-dns $(TEMP_DIR)
 	cp $(KUBE_ROOT)/build/kube-dns/Dockerfile $(TEMP_DIR)
 
 	# Replace BASEIMAGE with the real base image

```
###  Step 7: Build kube-dns image

	cd /<source_root>/kubernetes/build/kube-dns 
    make ARCH=s390x

### Step 8: Test the kube-dns image (Optional)

####Run kube-dns in Kubernetes pod

* Follow steps mentioned [here] (https://github.com/linux-on-ibm-z/docs/wiki/Building-Kubernetes) from "Running Kubernetes on a single host machine" section to start the cluster.
* Create file busybox.yaml with the following contents

         apiVersion: v1
         kind: Pod
         metadata:
           name: busybox
           namespace: default
         spec:
           containers:
           - image: s390x/busybox
             command:
               - sleep
               - "3600"
             imagePullPolicy: IfNotPresent
             name: busybox
           restartPolicy: Always
    
  _**NOTE:** kubectl binary and busybox.yaml should be in the same folder_

* Create pod using the below command

        kubectl create -f busybox.yaml

* Wait for this pod to go into the running state

       You can get pods status using the below command

         kubectl get pods busybox

       You should see:

         NAME      READY     STATUS    RESTARTS   AGE
         busybox   1/1       Running   0          <some-time>

* Validate DNS works

       Once that pod is running, excute `kubectl get services kubernetes`. The below output is seen:

          NAME         CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
          kubernetes   10.0.0.1     <none>        443/TCP   29m

### References:
http://kubernetes.io/docs/admin/dns/
