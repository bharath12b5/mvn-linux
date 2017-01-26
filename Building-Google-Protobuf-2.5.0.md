**Protocol Buffers** is a method of serializing structured data. It is useful in developing programs to communicate with each other over a wire or for storing data. The method involves an interface description language that describes the structure of some data and a program that generates source code from that description for generating or parsing a stream of bytes that represents the structured data.

_**General notes:**_ 

* When following the steps below please use a standard permission user unless otherwise specified.  
* A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

###Prerequisites
   1. For RHEL 6.8 and SLES 11-SP4 you will need to build Gcc 4.8+
      
      _**Note:** The gcc version available on the RHEL 6.8 and SLES 11-SP4 package repositories is too low level. It needs to be at least at version 4.8, so build and install gcc yourself by following these steps._
	  
     * Download and extract gcc
		
       ```shell
       cd /<source_root>/
       wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.bz2
       bunzip2 gcc-4.8.2.tar.bz2
       tar xvf gcc-4.8.2.tar
       cd gcc-4.8.2/
       ```

     * Automatically download all missing prerequisites

       ```shell
       ./contrib/download_prerequisites
       ```

     * Create a build directory and configure gcc to build

       ```shell
      cd /<source_root>/
      mkdir gccbuild
      cd gccbuild/
      ../gcc-4.8.2/configure --prefix=$HOME/install/gcc-4.8.2 --enable-shared --disable-multilib --enable-threads=posix --with-system-zlib --enable-languages=c,c++
      ```
      _**Note:** You can change the `-prefix=$HOME/install/gcc-4.8.2` directory to be a different installation location if you prefer, however please note the path entry will need changing later_

    * Build and install gcc

      ```shell
      make
      make install
      ```

    * Update the path to match the new gcc version

      ```shell
      export PATH=$HOME/install/gcc-4.8.2/bin:$PATH
      ```

      _**Note:** If you changed the directory in `configure`, ensure it matches above._

###Building Google Protobuf 2.5.0

1. Download Protobuf source code:

    ```shell
	cd /<source_root>/
    wget https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz
    tar zxvf protobuf-2.5.0.tar.gz
    cd protobuf-2.5.0
    ```

2. Retrieve a generic header file from the official repository (available since version 2.6.0): 

    ```shell
    wget https://raw.githubusercontent.com/google/protobuf/master/src/google/protobuf/stubs/atomicops_internals_generic_gcc.h -P src/google/protobuf/stubs/ 
    ```

3. Edit file `src/google/protobuf/stubs/atomicops.h`, navigate to `line 184`, add the following lines:

    ```c
    #elif defined(GOOGLE_PROTOBUF_ARCH_S390)
    #include <google/protobuf/stubs/atomicops_internals_generic_gcc.h>
    ```

4. Edit file `protobuf-2.5.0/src/google/protobuf/stubs/platform_macros.h`, navigate to `line 59`, add the following lines:

    ```c
    #elif defined(__s390x__)
    #define GOOGLE_PROTOBUF_ARCH_S390 1
    #define GOOGLE_PROTOBUF_ARCH_64_BIT 1
    ```
    
5. Configure the program, and run GNU make to build and install the binaries (please consult official build documentation for detailed configuration https://github.com/google/protobuf/blob/master/INSTALL.txt):

    ```shell
    ./configure
    make
    make check
    make install
    ```

6. Verify the installation

    ```shell
    export LD_LIBRARY_PATH=/usr/local/lib
    protoc --version
    ```
	_**Note:** Protobuf should report version `libprotoc 2.5.0`_