**Protocol Buffers** is a method of serializing structured data. It is useful in developing programs to communicate with each other over a wire or for storing data. The method involves an interface description language that describes the structure of some data and a program that generates source code from that description for generating or parsing a stream of bytes that represents the structured data.

Building Google Protobuf 2.5.0

1. Download Protobuf source code:

    ```shell
    wget https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz
    tar zxvf protobuf-2.5.0.tar.gz
    cd protobuf-2.5.0
    ```

2. Retrieve a generic header file from the official repository: 

    ```shell
    wget https://raw.githubusercontent.com/google/protobuf/master/src/google/protobuf/stubs/atomicops_internals_generic_gcc.h src/google/protobuf/stubs/atomicops_internals_generic_gcc.h 
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