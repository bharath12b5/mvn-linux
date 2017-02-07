<!---PACKAGE:Couchbase--->
<!---DISTRO:SLES 12.x:4.1.0--->
<!---DISTRO:RHEL 7.x:4.1.0--->
<!---DISTRO:Ubuntu 16.x:4.1.0--->

# Building Couchbase

The instructions provided below specify the steps to build Couchbase version 4.5.0 on IBM z Systems for following distributions:

* RHEL (6.8, 7.1, 7.2, 7.3)
* SLES (12, 12 SP1, 12 SP2)
* Ubuntu (16.04, 16.10)

_**General Notes:**_  

* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

#### Step 1 : Install the Dependencies

*	RHEL 6.8 
 
		sudo yum install wget tar git gcc-c++ make curl openssl-devel libcurl-devel snappy-devel ncurses-devel java-1.7.1-ibm java-1.7.1-ibm-devel openssl unixODBC unixODBC-devel cmake  libevent  libtool bunzip2 git-svn autoconf libbz2-devel bzip2-devel	

*	RHEL (7.1 ,7.2, 7.3)
     
		sudo yum install wget tar git gcc-c++ make curl openssl-devel libevent-devel libcurl-devel snappy-devel java-1.7.1-ibm java-1.7.1-ibm-devel ncurses-devel openssl unixODBC unixODBC-devel cmake subversion autoconf libbz2-devel bzip2-devel


    
*	SLES (12, 12 SP1, 12 SP2)

        sudo zypper install cmake wget tar git gcc-c++ make curl libopenssl-devel java-1.7.1-ibm-devel libevent-devel libcurl-devel snappy-devel ncurses-devel openssl unixODBC unixODBC-devel python-xml python-pyOpenSSL subversion autoconf libbz2-devel bzip2-devel	
        
       
*   Ubuntu (16.04, 16.10)
 
        sudo apt-get install git subversion make tar wget python gcc-4.8 g++-4.8 curl libssl-dev libevent-dev libcurl4-openssl-dev libsnappy-dev ncurses-dev openssl cmake autoconf libncurses-dev unixodbc unixodbc-dev libssl-dev
        sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 40
        sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.8 40
        sudo update-alternatives --config gcc
        sudo update-alternatives --config g++


	Install java using the below link **(Ubuntu 16.04/16.10 Only)**
    
	Download the Java SDK 8 Installable package for Linux on z Systems 64-bit from [here](https://developer.ibm.com/javasdk/downloads/#tab_sdk8) and use the command below to install java  
		
		chmod +x ibm-java-s390x-sdk-8.0-3.20.bin
		sudo ./ibm-java-s390x-sdk-8.0-3.20.bin	
		export JAVA_HOME=<path_to_java_installation_directory>
		export PATH=$JAVA_HOME/bin:$PATH

* For RHEL 6.8, install following additional dependencies:

    * **gcc**(4.8.2)

      ```bash
      wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.bz2
      bunzip2  gcc-4.8.2.tar.bz2
      tar xvf gcc-4.8.2.tar
      cd gcc-4.8.2
      ./contrib/download_prerequisites
      mkdir build
      cd build/
      ../configure --disable-multilib --disable-checking --enable-languages=c,c++ --enable-multiarch --enable-shared --enable-threads=posix --without-included-gettext --with-system-zlib --prefix=/opt/gcc4.8
      make && sudo make install
      export PATH=/opt/gcc4.8/bin:$PATH
      export LD_LIBRARY_PATH=/opt/gcc4.8/lib64/:$LD_LIBRARY_PATH  
      sudo sh -c "echo '/opt/gccgo/lib64' >> /etc/ld.so.conf.d/gccgo.conf"  
      sudo /sbin/ldconfig  
      gcc --version
      ```

    * **git**(2.0.0)

      ```bash
      sudo yum install -y curl-devel expat-devel gettext-devel openssl-devel perl-devel zlib-devel  
      wget https://www.kernel.org/pub/software/scm/git/git-2.0.0.tar.gz  
      tar xvzf git-2.0.0.tar.gz  
      cd git-2.0.0  
      ./configure --prefix=/usr && make && sudo make install  
      git --version  
      ```

    * **libevent**(2.0.22)

      ```bash
      git clone https://github.com/libevent/libevent.git
      cd libevent/
      git checkout release-2.0.22-stable
      ACLOCAL="aclocal -I /usr/share/aclocal" autoreconf -if
      ./autogen.sh && ./configure && make && sudo make install
      ```
		
**Other dependencies**

  *	To install Go, please refer to the [Go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7) recipe

  *  To install Python 2.7.12, follow the steps given below for RHEL 6.8, SLES(12, 12 SP1, 12 SP2)
     
      ```bash

       sudo yum install -y gcc gcc-c++ make ncurses patch xz xz-devel wget tar
       wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tar.xz
       tar -xvf Python-2.7.12.tar.xz
       cd Python-2.7.12
       ./configure --prefix=/usr/local --exec-prefix=/usr/local
       make && sudo make install
       sudo mv /usr/bin/python /usr/bin/python_old
       sudo ln -s <python-2.7.12-build-location>/bin/python /usr/bin/  
       python -V  
      ```    
      _**Note:** ```python -V``` should show ```2.7.12```._



  *   To install Erlang 17.4, follow the steps given below:
   
      ```bash
	  wget http://www.erlang.org/download/otp_src_17.4.tar.gz
	  tar zxvf otp_src_17.4.tar.gz
	  cd /<source_root>/otp_src_17.4
	  export ERL_TOP=`pwd`
	  ./configure --prefix=/usr
	  make
	  sudo make install
      ```	  
  * To install CMake 3.5 (Only for RHEL & SLES distributions), follow the steps given below:
   
      ```bash
      wget https://cmake.org/files/v3.5/cmake-3.5.2.tar.gz && \
      tar xzf cmake-3.5.2.tar.gz && \
      cd cmake-3.5.2 && \
      ./configure --prefix=/opt/cmake && \
      make && \
      sudo make install && \
      export PATH=$PATH:/opt/cmake/bin 
      ```
    
  *   To install V8 libraries, follow the steps given below:
   
    ```bash
	repo_url="https://github.com/ibmruntimes/v8z.git"
    repo_branch="4.5-s390"
    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git depot_tools
    export PATH=`pwd`/depot_tools:"$PATH"
    gclient config --spec "solutions = [{ \
    'name': 'v8', \
    'url': '$repo_url@$repo_branch' \
    }]"
    gclient sync
    cd v8
    make -j4 s390x GYPFLAGS="-Dv8_use_external_startup_data=0" library=shared i18nsupport=off
    make -j4 s390x GYPFLAGS="-Dv8_use_external_startup_data=0 -Dcomponent=shared_library"  i18nsupport=off
	sudo cp -vR include/* /usr/include/
	sudo chmod 644 /usr/include/libplatform/libplatform.h
	sudo chmod 644 /usr/include/v8*h
	```
    
    * For RHEL/SLES
    ```
	sudo cp -v out/s390x.release/lib.target/lib*.so /usr/local/lib64/ 
	sudo chmod -f 755 /usr/local/lib64/libv8.so
	sudo cp -v out/s390x.release/obj.target/tools/gyp/lib*.a /usr/local/lib64/
    ```
    
    * For Ubuntu
    ```
	sudo cp -v out/s390x.release/lib.target/lib*.so /usr/local/lib/ 
	sudo chmod -f 755 /usr/local/lib/libv8.so
	sudo cp -v out/s390x.release/obj.target/tools/gyp/lib*.a /usr/local/lib/
    ```
    
  * To install ICU, follow the steps given below:
  
    ```bash
    svn export http://source.icu-project.org/repos/icu/tags/release-54-1/ icu
    cd <source>/icu/icu4c/source
    ./configure --enable-static --prefix=/usr && make && sudo make install
    ```
    
    * For RHEL/SLES
    ```
    sudo cp /usr/lib/libicu*  /usr/local/lib64
    ```
    
    * For Ubuntu
    ```
    sudo cp /usr/lib/libicu*  /usr/local/lib
    ```
	
  * Install Flatbuffers
    
     *  Set environment variables (For RHEL 6.8 only)

    ```bash    
    export CMAKE_GENERATOR_CC=/opt/gcc4.8/bin/gcc 
    export LD_LIBRARY_PATH=/opt/gcc4.8/lib64/:$LD_LIBRARY_PATH 
    export GOROOT=/home/test/go 
    export CMAKE_INCLUDE_PATH=/opt/gccgo/lib64 
    export CMAKE_CXX_COMPILER=/opt/gcc4.8/bin/gcc 
    export CC=/opt/gcc4.8/bin/gcc 
    export CMAKE_C_COMPILER=/opt/gcc4.8/bin/gcc 
    ```
    *   Download the source code
    ```bash
    git clone https://github.com/google/flatbuffers
    cd flatbuffers
    ```
    *  Edit the file `include/flatbuffers/flatbuffers.h`

    ```diff
@@ -69,6 +69,7 @@

 // The wire format uses a little endian encoding (since that's efficient for
 // the common platforms).
+#define FLATBUFFERS_LITTLEENDIAN 0
 #if !defined(FLATBUFFERS_LITTLEENDIAN)
   #if defined(__GNUC__) || defined(__clang__)
     #ifdef __BIG_ENDIAN__
     ```
    
   *  Install Flatbuffers 

    ```bash
    cmake -G "Unix Makefiles"
    make
    export PATH=$PATH:`pwd` 
    sudo make install
    ```

		
#### Step 2 : Download the Repo tool

*   Download the Repo tool using cURL tool and ensure that it has execute permissions
        
     ```bash
		cd /<source_root>/
		curl https://storage.googleapis.com/git-repo-downloads/repo > repo
		chmod a+x repo
      ```	
            
#### Step 3 : Build, install and test Couchbase

*	Create a directory couchbase

    ```bash
	mkdir -p /<source_root>/couchbase
    ```
		
*	Clone the Couchbase 4.5.0 release using the the manifest file via Repo tool with the _init_ and _sync_ commands
	
       ```bash	
		cd /<source_root>/couchbase
		git config --global user.email your@email.addr
		git config --global user.name  your_name
		../repo init -u git://github.com/couchbase/manifest -m released/4.5.0.xml
        ../repo sync
       ```
		
*   Edit the following files
   
1. `/<source_root>/couchbase/tlm/cmake/Modules/CouchbaseDefinitions.cmake`
	
 ```diff
@@ -2,6 +2,7 @@ ADD_DEFINITIONS(-D_POSIX_PTHREAD_SEMANTICS)
ADD_DEFINITIONS(-D_GNU_SOURCE=1)
ADD_DEFINITIONS(-D__EXTENSIONS__=1)
ADD_DEFINITIONS(-D__STDC_FORMAT_MACROS)
+ADD_DEFINITIONS(-DWORDS_BIGENDIAN=1)

IF (NOT "" STREQUAL "${BUILD_ENTERPRISE}")
ADD_DEFINITIONS(-DENTERPRISE_EDITION=1)
```
 	
2. `/<source_root>/couchbase/forestdb/src/arch.h`
	
 ```diff
@@ -315,6 +315,10 @@
         #define spin_unlock(arg) pthread_spin_unlock(arg)
         #define spin_destroy(arg) pthread_spin_destroy(arg)
         #define SPIN_INITIALIZER (spin_t)(1)
+     #if defined(__s390x__)
+        #undef SPIN_INITIALIZER
+        #define SPIN_INITIALIZER (spin_t)(0)
+     #endif
     #endif
     #ifndef mutex_t
         // mutex
 ```
			             
3. `/<source_root>/couchbase/couchstore/src/views/bin/couch_view_file_merger.c`

 ```diff
@@ -35,7 +35,7 @@
     MERGE_FILE_TYPE_ID_BTREE = 'i',
     MERGE_FILE_TYPE_MAPREDUCE_VIEW = 'v',
     MERGE_FILE_TYPE_SPATIAL = 's'
-} merge_file_type_t;
+}; typedef unsigned char merge_file_type_t;


 int main(int argc, char *argv[])
 ```

4. `/<source_root>/couchbase/forestdb/utils/debug.cc`

 ```diff
@@ -89,6 +89,8 @@ static void sigsegv_handler(int sig, siginfo_t *siginfo, void *context)
     ucontext *u = (ucontext *)context;
 #ifdef REG_RIP // Test if the Program Counter is 64 bits
     unsigned char *pc = (unsigned char *)u->uc_mcontext.gregs[REG_RIP];
+#elif __s390x__
+    unsigned char *pc = (unsigned char *)u->uc_mcontext.psw.addr;
 #else // 32 bit machine, PC is stored in %eip register
     unsigned char *pc = (unsigned char *)u->uc_mcontext.gregs[REG_EIP];
 #endif // REG_RIP for 64-bit machines

 ```	
5. `/<source_root>/couchbase/platform/CMakeLists.txt`

 ```diff
@@ -134,7 +134,6 @@ ADD_LIBRARY(platform SHARED ${PLATFORM_FILES}
                             src/cbassert.c
                             src/checked_snprintf.cc
                             src/crc32c.cc
-                            src/crc32c_sse4_2.cc
                             src/crc32c_private.h
                             src/strerror.cc
                             src/thread.cc

 ```

6. `/<source_root>/couchbase/platform/include/platform/crc32c.h `
 
 ```diff
@@ -39,7 +39,7 @@
 // To fix will require refactoring to hide the X86 dependencies when
 // built on another platform.
 //
-#if !defined(__x86_64__) && !defined(_M_X64) && !defined(_M_IX86)
+#if !defined(__x86_64__) && !defined(_M_X64) && !defined(_M_IX86) && !defined(__s390x__)
 #error "crc32c requires X86 SSE4.2 for hardware acceleration"
 #endif

 ```

7. `/<source_root>/couchbase/platform/src/crc32c.cc `

 ```diff
 @@ -52,6 +52,7 @@
 //

 #include "platform/crc32c.h"
+#include <crc32-s390x.h>
 #include "crc32c_private.h"

 #include <stdint.h>
@@ -60,7 +61,7 @@
 // select header file for cpuid.
 #if defined(WIN32)
 #include <intrin.h>
-#elif defined(__clang__) || defined(__GNUC__)
+#elif defined(__clang__) || defined(__GNUC__) && !defined(__s390x__)
 #include <cpuid.h>
 #endif

@@ -370,7 +371,7 @@ crc32c_function setup_crc32c() {
     const uint32_t SSE42 = 0x00100000;

     crc32c_function f = crc32c_sw;
-
+/*
 #if defined(WIN32)
     std::array<int, 4> registers = {{0,0,0,0}};
     __cpuid(registers.data(), 1);
@@ -382,7 +383,7 @@ crc32c_function setup_crc32c() {
     if (registers[2] & SSE42) {
         f = crc32c_hw;
     }
-
+*/
     return f;
 }

@@ -397,4 +398,4 @@ PLATFORM_PUBLIC_API
 uint32_t crc32c (const uint8_t* buf, size_t len, uint32_t crc_in) {
     return safe_crc32c(buf, len, crc_in);
 }
-}
\ No newline at end of file
+}

 ```

8. `/<source_root>/couchbase/platform/tests/CMakeLists.txt `
 ```diff
 @@ -6,7 +6,7 @@ ADD_SUBDIRECTORY(backtrace)
 ADD_SUBDIRECTORY(base64)
 ADD_SUBDIRECTORY(checked_snprintf)
 ADD_SUBDIRECTORY(cjson)
-ADD_SUBDIRECTORY(crc32)
+#ADD_SUBDIRECTORY(crc32)
 ADD_SUBDIRECTORY(dirutils)
 ADD_SUBDIRECTORY(gethrtime)
 ADD_SUBDIRECTORY(gettimeofday)
 ```

9. `/<source_root>/couchbase/goproj/src/github.com/couchbase/indexing/secondary/memdb/skiplist/skiplist.go `
 ```diff
@@ -74,12 +74,13 @@ func NewWithConfig(cfg Config) *Skiplist {
        }

        if cfg.UseMemoryMgmt {
-               s.freeNode = func(n *Node) {
-                       if Debug {
-                               debugMarkFree(n)
-                       }
-                       cfg.Free(unsafe.Pointer(n))
-               }
+                s.freeNode = func(*Node) {}
+               //s.freeNode = func(n *Node) {
+                       //if Debug {
+                               //debugMarkFree(n)
+                       //}
+                       //cfg.Free(unsafe.Pointer(n))
+       //      }
        } else {
                s.freeNode = func(*Node) {}
        }
 ```

10. `/<source_root>/couchbase/memcached/include/libgreenstack/Message.h`
 ```diff
@@ -187,6 +187,7 @@ namespace Greenstack {
         /**
          * The flags in the message
          */
+#if !defined(__s390x__)
         struct Flags {
             bool response
                 : 1;
@@ -203,6 +204,24 @@ namespace Greenstack {
             bool next
                 : 1;
         } flags;
+#else
+        struct Flags {
+            bool next
+                : 1;
+            bool unassigned
+                : 2;
+            bool quiet
+                : 1;
+            bool more
+                : 1;
+            bool fence
+                : 1;
+            bool flexHeaderPresent
+                : 1;
+            bool response
+                : 1;
+        } flags;
+#endif

         /**
          * The flex header in the message

 ```

11. `/<source_root>/couchbase/memcached/daemon/subdocument_validators.cc`
  ```diff

@@ -144,8 +144,8 @@ is_valid_multipath_spec(const char* ptr, const SubdocMultiCmdTraits traits,
         headerlen = sizeof(*spec);
         opcode = protocol_binary_command(spec->opcode);
         flags = protocol_binary_subdoc_flag(spec->flags);
-        pathlen = ntohs(spec->pathlen);
-        valuelen = ntohl(spec->valuelen);
+        pathlen = ntohs(__bswap_16(spec->pathlen)); //SHS
+        valuelen = ntohl(__bswap_32(spec->valuelen)); //SHS

     } else {
         auto* spec = reinterpret_cast<const protocol_binary_subdoc_multi_lookup_spec*>
@@ -206,7 +206,7 @@ subdoc_multi_validator(void* packet, const SubdocMultiCmdTraits traits)
     // 1. Check simple static values.

     // Must have at least one lookup spec
-    const size_t minimum_body_len = ntohs(req->request.keylen) + req->request.extlen + traits.min_value_len;
+    const size_t minimum_body_len = ntohs(__bswap_16(req->request.keylen)) + req->request.extlen + traits.min_value_len; //SHS

     if ((req->request.magic != PROTOCOL_BINARY_REQ) ||
         (req->request.keylen == 0) ||
@@ -231,8 +231,8 @@ subdoc_multi_validator(void* packet, const SubdocMultiCmdTraits traits)
     // 2. Check that the lookup operation specs are valid.
     const char* const body_ptr = reinterpret_cast<const char*>(packet) +
                                  sizeof(*req);
-    const size_t keylen = ntohs(req->request.keylen);
-    const size_t bodylen = ntohl(req->request.bodylen);
+    const size_t keylen = ntohs(__bswap_16(req->request.keylen)); //SHS
+    const size_t bodylen = ntohl(__bswap_32(req->request.bodylen)); //SHS

  ```

12. `/<source_root>/couchbase/memcached/utilities/subdoc_encoder.cc`
 ```diff

@@ -66,8 +66,8 @@ std::vector<char> SubdocMultiMutationCmd::encode() const {
         } encoded;
         encoded.spec.opcode = s.opcode;
         encoded.spec.flags = s.flags;
-        encoded.spec.pathlen = htons(s.path.size());
-        encoded.spec.valuelen = htonl(s.value.size());
+        encoded.spec.pathlen = htons(__bswap_16(s.path.size())); //SHS
+        encoded.spec.valuelen = htonl(__bswap_32(s.value.size())); //SHS

         std::copy(&encoded.bytes[0],
                   &encoded.bytes[sizeof(encoded.spec)],
@@ -115,11 +115,11 @@ void SubdocMultiCmd::populate_header(protocol_binary_request_header& header,
                                      size_t bodylen) const {
     header.request.magic = PROTOCOL_BINARY_REQ;
     header.request.opcode = command;
-    header.request.keylen = htons(key.size());
+    header.request.keylen = htons(__bswap_16(key.size())); //SHS
     header.request.extlen = (expiry != 0 || encode_zero_expiry_on_wire) ? sizeof(uint32_t) : 0;
     header.request.datatype = PROTOCOL_BINARY_RAW_BYTES;
     /* TODO: vbucket */
-    header.request.bodylen = htonl(bodylen);
+    header.request.bodylen = htonl(__bswap_32(bodylen)); //SHS
     header.request.opaque = 0xdeadbeef;
     header.request.cas = cas;
 }
```
*   Replace Boltdb package

  ```bash
   cd ./godeps/src/github.com/
   mv boltdb boltdb_ORIG
   mkdir boltdb
   cd boltdb
   git clone https://github.com/boltdb/bolt.git
   cd bolt/
   git checkout v1.3.0

  ```
*  Add the  s390x crc32 support to the source code.

    ```
    git clone https://github.com/linux-on-ibm-z/crc32-s390x
    cd crc32-s390x
    export CC=/usr/bin/gcc (For RHEL6.8)
    make
    sudo cp crc32-s390x.h /usr/include/
    ```
    
    * For RHEL/SLES
    ```
    sudo cp libcrc32_s390x.a /usr/local/lib64/
    ```
    
    * For Ubuntu
    ```
    sudo cp libcrc32_s390x.a /usr/local/lib/
    ```
*   Set environment variables
  
   ```bash
     export CB_MULTI_GO=0
     export CC=/opt/gcc4.8/bin/gcc (For RHEL6.8)
   ```


*   Run the make command and start the building procedure
 
   ```bash
	cd /<source_root>/couchbase/
	make
   ```			
*   Run the test cases using ctest tool

    ```bash
	cd /<source_root>/couchbase/build/
	ctest
    ```    
    _*Note: There are some test case failures, which are also observed on Intel x86 VM. These failures can be ignored.*_

#### Step 5 : Start the server
			
*	Use the below command to start Couchbase Server

		/<source_root>/couchbase/install/bin/couchbase-server -- -noinput &
		
	 We can view the  Couchbase UI at   `http://localhost:8092`           
      


##### References:
http://www.couchbase.com/nosql-databases/couchbase-server
