<!---PACKAGE:Protobuf--->
<!---DISTRO:SLES 12:2.6.1--->
<!---DISTRO:SLES 11:2.6.1--->
<!---DISTRO:RHEL 7.1:2.6.1--->
<!---DISTRO:RHEL 6.6:2.6.1--->
<!---DISTRO:Ubuntu 16.x:2.6.1--->

# Building Protobuf

Below versions of Protobuf are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04     has `2.6.1`

The instructions provided below specify the steps to build Google Protobuf v2.6.1 on Linux on the IBM z Systems for RHEL 6/7 and SLES 11/12. (Protobuf is available at https://developers.google.com/protocol-buffers/ and the github repository can be found at https://github.com/google/protobuf/):

_**General notes:**_ 

i) When following the steps below please use a standard permission user unless otherwise specified.  
ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

## Building Protobuf 2.6.1 
	
1. Install the build time dependencies

  For **RHEL 7.1 / 6.6** use the following
  ```shell
  sudo yum install tar wget autoconf libtool automake gcc-c++ make git bzip2 curl unzip zlib zlib-devel
  ```
  For **SLES 12 / 11** (note the package name differences)
  ```shell
  sudo zypper install tar wget autoconf libtool automake gcc-c++ make git bzip2 curl unzip zlib zlib-devel
  ```
 
  You may already have these packages installed - just install any missing.

2. For RHEL 6.6 and SLES 11 you will need to build Gcc 4.8+ directly

  The gcc version available on the RHEL 6.6 and SLES 11 package repositories is too low level. It needs to be at least at version 4.8, so build and install gcc yourself by following these steps
  
    1. Download and extract gcc
    
    ```shell
    cd /<source_root>/
    wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.bz2
    bunzip2 gcc-4.8.2.tar.bz2
    tar xvf gcc-4.8.2.tar
    cd gcc-4.8.2/
    ```
    2. Automatically download all missing prerequisites
    
    ```shell
    ./contrib/download_prerequisites
    ```
    3. Create a build directory and configure gcc to build
    
    ```shell
    cd /<source_root>/
    mkdir gccbuild
    cd gccbuild/
    ../gcc-4.8.2/configure --prefix=$HOME/install/gcc-4.8.2 --enable-shared --disable-multilib --enable-threads=posix --with-system-zlib --enable-languages=c,c++
    ```
    _**Note:** You can change the `-prefix=$HOME/install/gcc-4.8.2` directory to be a different installation location if you prefer, however please note the path entry will need changing later_
    4. Build and install gcc
    
    ```shell
    make
    make install
    ```
    5. Update the path to match the new gcc version
    
    ```shell
    export PATH=$HOME/install/gcc-4.8.2/bin:$PATH
    ```
    _**Note:** If you changed the directory in step iii ensure it matches above_
3. Clone the protobuf repository and checkout the correct version

  ```shell
  cd /<source_root>/
  git clone https://github.com/google/protobuf.git
  cd protobuf
  git checkout v2.6.1
  ```
3. Generate and then run the configuration

  ```shell
  ./autogen.sh
  ./configure
  ```
4. Build and (_optionally_) test

  ```shell
  make
  make check
  ```
  _**Note:** There are 5 tests/suites.  All 5 should pass_
5. Install Protobuf and verify the installation

  ```shell
  sudo make install
  export LD_LIBRARY_PATH=/usr/local/lib
  protoc --version
  ```
  _**Note:** Protobuf should report version `libprotoc 2.6.1`_

# References
  https://developers.google.com/protocol-buffers/  
  https://github.com/google/protobuf/