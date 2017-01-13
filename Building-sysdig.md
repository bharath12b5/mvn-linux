<!---PACKAGE:sysdig--->
<!---DISTRO:SLES 12.x:0.13.0--->
<!---DISTRO:RHEL 7.x:0.13.0--->
<!---DISTRO:Ubuntu 16.x:0.13.0--->

# Building sysdig

Below versions of sysdig are available in respective distributions at the time of this recipe creation:
* Ubuntu 16.04 has `0.8.0`
* Ubuntu 16.10 has `0.9.0`

The instructions provided below specify the steps to build sysdig version 0.13.0 on IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_
* _When following the steps below please use a root user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##Step 1: Building and Installing sysdig
####1.1) Install dependencies

  * RHEL 7.1/7.2/7.3
    ```sh
    yum install wget tar gcc cmake gcc-c++ make lua-devel.s390x kernel-devel
    ```

  * SLES 12-SP1/12-SP2
    ```sh
    zypper install wget tar gcc cmake make gcc-c++ lua51 lua51-devel kernel-default-devel
    ```

  * Ubuntu 16.04/16.10
    ```sh
    apt-get install wget tar gcc cmake g++ lua5.1 lua5.1-dev linux-headers
    ```

_**Note:** For linux headers, multiple versions might be present, user can select latest version to install._

####1.2) Download source code
  ```sh
  cd  /<source_root>/
  wget https://github.com/draios/sysdig/archive/0.13.0.tar.gz
  tar -xvzf 0.13.0.tar.gz
  cd sysdig-0.13.0
  mkdir build
  ```

####1.3) Configure sysdig
  ```sh
  cd  /<source_root>/sysdig-0.13.0/build
  cmake -DUSE_BUNDLED_LUAJIT=OFF ..
  ```

####1.4) Change file `/<source_root>/sysdig-0.13.0/build/driver/Makefile.dkms` to update the Kernel directory.
```diff	
@@ -2,7 +2,7 @@
 obj-m += sysdig-probe.o
 ccflags-y :=

-KERNELDIR              ?= /lib/modules/$(shell uname -r)/build
+KERNELDIR              ?= /lib/modules/3.10.0-514.2.2.el7.s390x/build

 TOP := $(shell pwd)
 all:
```

_**Note:**_
* _Here `3.10.0-514.2.2.el7.s390x` is obtained by `uname -r`, this may vary according to the kernel available on your system._
* _`kernel_directory` is the one available in `/lib/modules/`. In case no `kernel_directory` is present in `/lib/modules` then create directory in `/lib/modules/` using `mkdir <dirname>`, dirname name should match with one which is present in `/usr/src` or `/usr/src/kernels` (RHEL7.1/7.2/7.3). Once kernel directory is created in `/lib/modules/` perform the below softlink:_

  * RHEL 7.1/7.2/7.3
    ```sh
    ln -s /usr/src/kernels/<kernel_directory> /lib/modules/<kernel_directory>/build
    ```

  * SLES 12-SP2/12-SP2 and Ubuntu 16.04/16.10
    ```sh
    ln -s /usr/src/<kernel_directory> /lib/modules/<kernel_directory>/build
    ```

####1.5) Build and Install sysdig
  ```sh
  cd  /<source_root>/sysdig-0.13.0/build/
  make
  make install
  ```

####1.6) Insert sysdig driver module
  ```sh
  cd  /<source_root>/sysdig-0.13.0/build/driver/
  insmod sysdig-probe.ko
  ```

## Reference:
http://www.sysdig.org/