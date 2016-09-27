<!---PACKAGE:Protobuf--->
<!---DISTRO:SLES 12:3.0.0--->
<!---DISTRO:SLES 11:3.0.0--->
<!---DISTRO:RHEL 7.1:3.0.0--->
<!---DISTRO:RHEL 6.6:3.0.0--->
<!---DISTRO:Ubuntu 16.x:3.0.0--->

# Building Protobuf

Below versions of Protobuf are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04     has `2.6.1`

The instructions provided below specify the steps to build Google Protobuf v3.0.0 on Linux on the IBM z Systems for RHEL 6/7 and SLES 11/12 and Ubuntu 16.04. (Protobuf is available at https://developers.google.com/protocol-buffers/ and the github repository can be found at https://github.com/google/protobuf/):

_**General notes:**_ 

i) When following the steps below please use a standard permission user unless otherwise specified.  
ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

## Building Protobuf 3.0.0 
	
1. Install the build time dependencies

  For **RHEL 7.1 / 6.6** use the following
  ```shell
  sudo yum install tar wget autoconf libtool automake gcc-c++ make git bzip2 curl unzip zlib zlib-devel
  ```
  For **SLES 12 / 11** (note the package name differences)
  ```shell
  sudo zypper install tar wget autoconf libtool automake gcc-c++ make git bzip2 curl unzip zlib zlib-devel
  ```
  For **Ubuntu 16.04**
  ```shell
  sudo apt-get install tar wget autoconf libtool automake g++ make git bzip2 curl unzip zlib1g-dev
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
  git checkout v3.0.0
  ```
4. The file `autogen.sh` contains obsolete link to Google Mock. Replace the contents as shown below.

   ```shell
   echo "Google Mock not present.  Fetching gmock-1.7.0 from the web..."
   curl $curlopts -O https://googlemock.googlecode.com/files/gmock-1.7.0.zip
   unzip -q gmock-1.7.0.zip
   rm gmock-1.7.0.zip
   mv gmock-1.7.0 gmock  
   ```
   
   Change the above lines to:

   ```shell 
   echo "Google Mock not present.  Fetching gmock-1.7.0 from the web..."
   curl $curlopts -L -O https://github.com/google/googlemock/archive/release-1.7.0.zip
   unzip -q release-1.7.0.zip
   rm release-1.7.0.zip
   mv googlemock-release-1.7.0 gmock

   curl $curlopts -L -O https://github.com/google/googletest/archive/release-1.7.0.zip
   unzip -q release-1.7.0.zip
   rm release-1.7.0.zip
   mv googletest-release-1.7.0 gmock/gtest
   ```
5. Generate and then run the configuration

  ```shell
  ./autogen.sh
  ./configure
  ```

6. Add the below content to file `/<source_root>/protobuf/src/google/protobuf/stubs/atomicops_internals_generic_gcc.h` above `#endif // defined(__LP64__)` at line 131.

    ```shell
    inline Atomic64 NoBarrier_AtomicIncrement(volatile Atomic64* ptr,
                                         Atomic64 increment) {
    return __atomic_add_fetch(ptr, increment, __ATOMIC_RELAXED);
    }

    inline void NoBarrier_Store(volatile Atomic64* ptr, Atomic64 value) {
    __atomic_store_n(ptr, value, __ATOMIC_RELAXED);
    }

    inline Atomic64 NoBarrier_AtomicExchange(volatile Atomic64* ptr,
                                         Atomic64 new_value) {
    return __atomic_exchange_n(ptr, new_value, __ATOMIC_RELAXED);
    }

    inline Atomic64 NoBarrier_Load(volatile const Atomic64* ptr) {
    return __atomic_load_n(ptr, __ATOMIC_RELAXED);
    }
	```
  
7. Build and (_optionally_) test

  ```shell
  make
  make check
  ```
  _**Note:** There are 7 tests/suites.  All 7 should pass_

8. Install Protobuf and verify the installation

  ```shell
  sudo make install
  export LD_LIBRARY_PATH=/usr/local/lib
  protoc --version
  ```
  _**Note:** Protobuf should report version `libproto 3.0.0`_

# References
  https://developers.google.com/protocol-buffers/  
  https://github.com/google/protobuf/