<!---PACKAGE:TensorFlow--->
<!---DISTRO:SLES 12.x:0.10.0--->
<!---DISTRO:RHEL 7.x:0.10.0--->
<!---DISTRO:Ubuntu 16.x:0.10.0--->

# Building TensorFlow

The instructions provided below specify the steps to build TensorFlow version 0.10.0 on IBM z Systems for 
RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10.  


### _**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._


##Step 1: Building and Installing TensorFlow v0.10.0
 
####1.1) Install the dependencies

  * RHEL 7.1/7.2/7.3
  ```shell
   sudo yum install -y wget tar make flex zip unzip git vim gcc gcc-c++ libstdc++-* binutils-devel bzip2 which java-1.8.0-openjdk-devel.s390x automake autoconf libtool zlib  pkgconfig zlib-devel curl bison glibc* python-setuptools python-setuptools-devel python-devel swig numpy libcurl-devel scipy
sudo easy_install pip
sudo pip install wheel mock
  ```

  * SLES 12-SP1/12-SP2
  ```shell
   sudo zypper install -y wget tar make flex zip unzip git vim gcc gcc-c++ binutils-devel bzip2 glibc-devel glibc* makeinfo zlib-devel curl which java-1_8_0-openjdk-devel automake autoconf libtool zlib  pkg-config python-setuptools python-devel swig libcurl-devel python-numpy-devel 
sudo easy_install pip
sudo pip install wheel mock scipy
  ```  
  
  * Ubuntu 16.04/16.10
  ```shell
  sudo apt-get update
  sudo apt-get install -y pkg-config zip g++ zlib1g-dev unzip git vim tar wget automake autoconf libtool make curl maven openjdk-8-jdk python-pip python-virtualenv python-numpy swig python-dev libcurl3-dev python-mock python-scipy bzip2 glibc* 
sudo easy_install pip
sudo pip install wheel

  ```

####1.2)  Install maven ( For RHEL and SLES only)   

* Download maven binary
  
  ```shell  
  cd /<source_root>/
  wget http://www.eu.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
  tar -xvzf apache-maven-3.3.9-bin.tar.gz
  export PATH=$PATH:/<source_root>/apache-maven-3.3.9/bin/
  ```  

####1.3)  Build Protobuf

* Download source code of Protobuf

  ```shell
  cd /<source_root>/    
  git clone https://github.com/google/protobuf.git
  cd protobuf
  git checkout v3.0.0-beta-2
  ```

* Configure and build  

  ```shell
  ./autogen.sh
  LDFLAGS="-static" ./configure && \
  sed -i "s/^LDFLAGS = -static/LDFLAGS = -all-static/g" src/Makefile 
  make
  sudo make install 
  export LD_LIBRARY_PATH=/usr/local/lib
  sudo ldconfig
  cd java
  mvn package
  ```  

 Protobuf executable will be created at location: `/<source_root>/protobuf/src/`  
 Protobuf-java jar will be created at location: `/<source_root>/bazel/third_party/protobuf/`  


 _**Note:** If `./autogen.sh` fails with `cannot find zipfile directory in one of gmock-1.7.0.zip` error, apply below patch in file `autogen.sh`._
   ```diff
   @@ -31,10 +31,14 @@ fi
  # directory is set up as an SVN external.
  if test ! -e gmock; then
    echo "Google Mock not present.  Fetching gmock-1.7.0 from the web..."
 -  curl $curlopts -O https://googlemock.googlecode.com/files/gmock-1.7.0.zip
 -  unzip -q gmock-1.7.0.zip
 -  rm gmock-1.7.0.zip
 -  mv gmock-1.7.0 gmock
 +  curl $curlopts -L -O https://github.com/google/googlemock/archive/release-1.7.0.zip
 +  unzip -q release-1.7.0.zip
 +  rm release-1.7.0.zip
 +  mv googlemock-release-1.7.0 gmock
 +  curl $curlopts -L -O https://github.com/google/googletest/archive/release-1.7.0.zip
 +  unzip -q release-1.7.0.zip
 +  rm release-1.7.0.zip
 +  mv googletest-release-1.7.0 gmock/gtest
  fi
  ```  

 _**Note:** While building Protobuf, if build fails with `undefined reference to google::protobuf::` errors, apply  below patch in file `src/google/protobuf/stubs/atomicops_internals_generic_gcc.h`:_

  ```diff
  @@ -128,6 +128,23 @@ inline Atomic64 NoBarrier_CompareAndSwap(volatile Atomic64* ptr,
   return old_value;
   }

   +inline Atomic64 NoBarrier_AtomicIncrement(volatile Atomic64* ptr,
   +                                          Atomic64 increment) {
   +  return __atomic_add_fetch(ptr, increment, __ATOMIC_RELAXED);
   +}
   +
   +inline void NoBarrier_Store(volatile Atomic64* ptr, Atomic64 value) {
   +  __atomic_store_n(ptr, value, __ATOMIC_RELAXED);
   +}
   +
   +inline Atomic64 NoBarrier_AtomicExchange(volatile Atomic64* ptr,
   +                                         Atomic64 new_value) {
   +  return __atomic_exchange_n(ptr, new_value, __ATOMIC_RELAXED);
   +}
   +
   +inline Atomic64 NoBarrier_Load(volatile const Atomic64* ptr) {
   +  return __atomic_load_n(ptr, __ATOMIC_RELAXED);
   +}
    #endif // defined(__LP64__)

    }  // namespace internal
  ```  

  _**Note:** If Protobuf configure fails with `error: C compiler cannot create executable`, run configure without LDFLAGS settings._  
  
####1.4)  Build Protobuf Java Lite

* Download source code of Protobuf Java Lite

  ```shell
  cd /<source_root>/
  mkdir protobuf_javalite
  cd protobuf_javalite
  git clone https://github.com/google/protobuf
  cd protobuf
  git checkout v3.0.0-javalite
  ```
 
* Configure and build 

 ```shell
  ./autogen.sh
  LDFLAGS="-static" ./configure && \
  sed -i "s/^LDFLAGS = -static/LDFLAGS = -all-static/g" src/Makefile 
  make
  sudo cp src/protoc-gen-javalite /usr/local/bin/
  ``` 
 
 Protobuf Java Lite executable will be created at location: `/<source_root>/protobuf_javalite/protobuf/src/`  


  _**Note:** If `./autogen.sh` fails with `cannot find zipfile directory in one of gmock-1.7.0.zip` error, apply below patch in file `autogen.sh`._
   ```diff
   @@ -31,10 +31,14 @@ fi
  # directory is set up as an SVN external.
  if test ! -e gmock; then
    echo "Google Mock not present.  Fetching gmock-1.7.0 from the web..."
 -  curl $curlopts -O https://googlemock.googlecode.com/files/gmock-1.7.0.zip
 -  unzip -q gmock-1.7.0.zip
 -  rm gmock-1.7.0.zip
 -  mv gmock-1.7.0 gmock
 +  curl $curlopts -L -O https://github.com/google/googlemock/archive/release-1.7.0.zip
 +  unzip -q release-1.7.0.zip
 +  rm release-1.7.0.zip
 +  mv googlemock-release-1.7.0 gmock
 +  curl $curlopts -L -O https://github.com/google/googletest/archive/release-1.7.0.zip
 +  unzip -q release-1.7.0.zip
 +  rm release-1.7.0.zip
 +  mv googletest-release-1.7.0 gmock/gtest
  fi
  ```  

 _**Note:** While building Protobuf, if build fails with `undefined reference to google::protobuf::` errors, apply  below patch in file `src/google/protobuf/stubs/atomicops_internals_generic_gcc.h`:_

  ```diff
  @@ -128,6 +128,23 @@ inline Atomic64 NoBarrier_CompareAndSwap(volatile Atomic64* ptr,
   return old_value;
   }

   +inline Atomic64 NoBarrier_AtomicIncrement(volatile Atomic64* ptr,
   +                                          Atomic64 increment) {
   +  return __atomic_add_fetch(ptr, increment, __ATOMIC_RELAXED);
   +}
   +
   +inline void NoBarrier_Store(volatile Atomic64* ptr, Atomic64 value) {
   +  __atomic_store_n(ptr, value, __ATOMIC_RELAXED);
   +}
   +
   +inline Atomic64 NoBarrier_AtomicExchange(volatile Atomic64* ptr,
   +                                         Atomic64 new_value) {
   +  return __atomic_exchange_n(ptr, new_value, __ATOMIC_RELAXED);
   +}
   +
   +inline Atomic64 NoBarrier_Load(volatile const Atomic64* ptr) {
   +  return __atomic_load_n(ptr, __ATOMIC_RELAXED);
   +}
    #endif // defined(__LP64__)

    }  // namespace internal
  ```  

   _**Note:** If Protobuf configure fails with `error: C compiler cannot create executable`, run configure without LDFLAGS settings._  
   
####1.5)  Build gRPC Java

* Download gRPC Java
		
  ```shell
  cd /<source_root>/
  git clone https://github.com/linux-on-ibm-z/grpc-java
  cd grpc-java
  git checkout 0.15.0-s390x
  ```
* Build gRPC Java

  ```shell
  sudo ./gradlew build -Pprotoc=/usr/local/bin/protoc -Pprotoc-gen-javalite=/usr/local/bin/protoc-gen-javalite -x grpc-interop-testing:test -x grpc-netty:test -x grpc-protobuf:test -x grpc-protobuf-nano:test -x grpc-okhttp:test -x grpc-benchmarks:test
  ```  

  gRPC Java executable will be created at location: `/<source_root>/grpc-java/compiler/build/artifacts/java_plugin/`   


 _**Note:** While building gRPC Java, if build fails with an error `grpc-compiler:linkJava_pluginExecutable/usr/bin/ld:cannot find -lstdc++ `, install `libstdc++`_  

####1.6)  Build Bazel

* Download Bazel 
  ```shell
   cd /<source_root>/
   git clone https://github.com/linux-on-ibm-z/bazel
   cd bazel
   git checkout 0.3.2-s390x  
  ```

* Copy required files to Bazel source code
		
  ```shell
  cp /<source_root>/protobuf/src/protoc /<source_root>/bazel/third_party/protobuf/protoc-linux-s390x_64.exe
  cp /<source_root>/protobuf/java/target/protobuf-java-3.0.0-beta-2.jar /<source_root>/bazel/third_party/protobuf/protobuf-java-3.0.0-beta-2.jar
  cp /<source_root>/grpc-java/compiler/build/artifacts/java_plugin/protoc-gen-grpc-java.exe /<source_root>/bazel/third_party/grpc/protoc-gen-grpc-java-0.15.0-linux-s390x_64.exe
  ```
* Build Bazel

  ```shell
  ./compile.sh 
  export PATH=$PATH:/<source_root>/bazel/output/
  ```  

  _**Note:** While building Bazel, if build fails with an error `java.lang.OutOfMemoryError: Java heap space `, add below java options in `scripts/bootstrap/compile.sh`:_  

  ```diff
  - run "${JAVAC}" -classpath "${classpath}" -sourcepath "${sourcepath}" \
  + run "${JAVAC}" -J-Xms1g -J-Xmx1g -classpath "${classpath}" -sourcepath "${sourcepath}" \
  ```  

####1.7)  Build GCC 5.4.0 ( For RHEL and SLES only)    
   _TensorFlow requires GCC 5.4.0 and above._  

* Download and extract gcc
		
  ```shell
  cd /<source_root>/
  wget ftp://gcc.gnu.org/pub/gcc/releases/gcc-5.4.0/gcc-5.4.0.tar.gz
  tar -xvzf gcc-5.4.0.tar.gz
  cd gcc-5.4.0/
  ./contrib/download_prerequisites
  ```
* Create a build directory and configure gcc to build

  ```shell
  cd /<source_root>/
  mkdir gccbuild
  cd gccbuild/  
  ```

 * RHEL 7.1/7.2/7.3
 ```shell
 ../gcc-5.4.0/configure --prefix="/opt/gcc" --enable-shared --with-system-zlib --enable-threads=posix --enable-__cxa_atexit --enable-checking --enable-gnu-indirect-function --enable-languages="c,c++" --disable-bootstrap --disable-multilib
  ```

 * SLES 12-SP1/12-SP2
 ```shell
../gcc-5.4.0/configure --prefix="/opt/gcc" --enable-shared --with-system-zlib --enable-threads=posix --enable-__cxa_atexit --enable-checking --enable-gnu-indirect-function --enable-languages="c,c++" --disable-bootstrap
  ```
 
 _**Note:** You can change the `-prefix="/opt/gcc"` directory to be a different installation location if you prefer, however please note the path entry will change later._

* Build and install gcc

  ```shell
  make
  sudo make install
  ```
* Update the path to match the new gcc version

  ```shell
  export PATH=/opt/gcc/bin:$PATH  
  ```  
 _**Note:** If you changed the directory in `configure`, ensure it matches above._  


####1.8)  Build TensorFlow

* Download TensorFlow
  ```shell
  cd /<source_root>/
  git clone https://github.com/linux-on-ibm-z/tensorflow
  cd tensorflow
  git checkout v0.10.0-s390x
  ```

* Configure (without GPU support)

  ```shell
  ./configure
  Please specify the location of python. [Default is /usr/bin/python]:
  Do you wish to build TensorFlow with Google Cloud Platform support? [y/N] y
  Google Cloud Platform support will be enabled for TensorFlow
  Do you wish to build TensorFlow with GPU support? [y/N] N
  No GPU support will be enabled for TensorFlow
  Configuration finished
  ```  

* Build TensorFlow
  ```shell  
  bazel build -c opt //tensorflow/tools/pip_package:build_pip_package
   ```  

 _**Note:** While building TensorFlow, if build fails with below errors, apply respective fixes:_  

 * Error: `version GLIBCXX_3.4.20 not found`, run below commands:  
    ```shell
     sudo sh -c "echo '/opt/gcc/lib64' >> /etc/ld.so.conf.d/gcc.conf"
     sudo /sbin/ldconfig
    ```  

 * Error: `unknown target CPU` while building BoringSSL module, add s390x support in `/<source_root>/tensorflow/bazel-tensorflow/external/boringssl_git/src/include/openssl/base.h`  
     ```diff           
     + #elif defined(__s390x__)  
     + #define OPENSSL_64_BIT  
       #else  
       #error "Unknown target CPU"  
       #endif  
      ```
           
 * If build fails with `undefined reference to google::protobuf::` errors, apply below patch in file `/<source_root>/tensorflow/bazel-tensorflow/external/protobuf/src/google/protobuf/stubs/atomicops_internals_generic_gcc.h`:   

     ```diff      
     @@ -128,6 +128,24 @@ inline Atomic64 NoBarrier_CompareAndSwap(volatile Atomic64* ptr, 
      return old_value; 
    } 
    +inline Atomic64 NoBarrier_AtomicIncrement(volatile Atomic64* ptr, 
    + Atomic64 increment) { 
    + return __atomic_add_fetch(ptr, increment, __ATOMIC_RELAXED); 
    +} 
    + 
    +inline void NoBarrier_Store(volatile Atomic64* ptr, Atomic64 value) { 
    + __atomic_store_n(ptr, value, __ATOMIC_RELAXED); 
    +} 
    + 
    +inline Atomic64 NoBarrier_AtomicExchange(volatile Atomic64* ptr, 
    + Atomic64 new_value) { 
    + return __atomic_exchange_n(ptr, new_value, __ATOMIC_RELAXED); 
    +} 
    + 
    +inline Atomic64 NoBarrier_Load(volatile const Atomic64* ptr) { 
    + return __atomic_load_n(ptr, __ATOMIC_RELAXED); 
    +}  
    #endif // defined(__LP64__)
   ```

 * Error: `./tensorflow/core/kernels/sparse_matmul_op.h:46:26: error: 'Packet4f' does not name a type `
 `EIGEN_DEVICE_FUNC inline Packet4f pexpand_bf16_l(const Packet4f& from) { `

   Use this [patch]( https://github.com/benoitsteiner/tensorflow/commit/579438b835aafc377bba01ef6c50beca896a56bb) in file `tensorflow/core/kernels/sparse_matmul_op.h` 
   

 * Error: Internal compiler killed/gcc crashed, build using following command to restrict amount of memory used during build:  
`bazel build -c opt --local_resources 2048,.5,1.0  //tensorflow/tools/pip_package:build_pip_package`  


####1.9)  Build TensorFlow wheel and Install TensorFlow

  ```shell
   bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_wheel
   sudo pip install /tmp/tensorflow_wheel/tensorflow-0.10.0-py2-none-any.whl
  ```  

##Step 2: Verify TensorFlow (Optional)
* Run TensorFlow from command Line 

  ```shell
   $ python
    >>> import tensorflow as tf
    >>> hello = tf.constant('Hello, TensorFlow!')
    >>> sess = tf.Session()
    >>> print(sess.run(hello))
        Hello, TensorFlow!
    >>> a = tf.constant(10)
    >>> b = tf.constant(32)
    >>> print(sess.run(a + b))
       42
    >>>

  ```  

##Step 3: Execute Test Suite (Optional)  

* Run complete testsuite  

  ```shell
  bazel test  -c opt //tensorflow/...
  ```
* Run individual test 
  ```shell
  bazel test  -c opt //tensorflow/<module_name>:<testcase_name>
  ```  
    For example,   
    ```shell
    bazel test  -c opt //tensorflow/python/kernel_tests:topk_op_test
    ```  


    _**Note:** If below tests fail, apply patch in file `bazel-tensorflow/external/farmhash_archive/farmhash-34c13ddfab0e35422f4c3979f360635a8c050260/src/farmhash.h` :_  

   `//tensorflow/contrib/layers:sparse_feature_cross_op_test`  
   `//tensorflow/contrib/learn:tensorflow_dataframe_test`  
   `//tensorflow/contrib/linear_optimizer:sdca_ops_test`  
   `//tensorflow/core:platform_fingerprint_test `    
   
     
  ```diff   
   }  // namespace NAMESPACE_FOR_HASH_FUNCTIONS
   
   +// Detect big-endian architectures and inject FARMHASH_BIG_ENDIAN flag.
   +#if defined (__mips__)
   +  #if defined (__MIPSEB__)
   +    #define FARMHASH_BIG_ENDIAN 
   +  #elif defined (__MIPSEL__)
   +  #else
   +    #error MIPS no endian specified
   +  #endif
   +#elif defined (__s390__) || defined (__s390x__)
   +  #define FARMHASH_BIG_ENDIAN 
   +#elif defined (__powerpc__) || defined(__powerpc64__)
   +  #if defined (__LITTLE_ENDIAN__)
   +  #else
   +    #define FARMHASH_BIG_ENDIAN 
   +  #endif
   +#elif defined (__hppa__)
   +  #define FARMHASH_BIG_ENDIAN 
   +#elif defined (__sparc__)
   +  #define FARMHASH_BIG_ENDIAN 
   +#elif defined (__alpha__)
   +  #define FARMHASH_BIG_ENDIAN 
   +#endif // end detect big-endian archs
   +
   #endif  // FARM_HASH_H_

   ```    

   o This [issue](https://github.com/google/farmhash/issues/10) has been already opened with FarmHash community.   
   
   _**Note:** Below test from python module is failing as it uses google/highwayhash which generates different hash results on IBM z Systems :_  
   `//tensorflow/python/kernel_tests:string_to_hash_bucket_op_test`   
 
    o This [issue](https://github.com/google/highwayhash/issues/35) has been already opened with HighwayHash community. As per the issue, below patch can be applied in file `bazel-tensorflow/external/highwayhash/highwayhash/sip_hash.h` to pass the test. 
     
  ```diff  
   + #if ARCH_X64
   + #define HH_LITTLE_ENDIAN 1
   + #define HH_BIG_ENDIAN 0
   + #else
   + #define HH_LITTLE_ENDIAN 0
   + #define HH_BIG_ENDIAN 1
   + #endif


   INLINE void Update(const char* bytes) {
      uint64 packet;
      memcpy(&packet, bytes, sizeof(packet));
  + #if HH_BIG_ENDIAN
  + 	packet = __builtin_bswap64(packet);
  + #endif
      v3 ^= packet;

   ```  

 
    _**Note:** Below tests are failing in v0.10.0 however passing on master:_  
   `//tensorflow/python/kernel_tests:determinant_op_test`  
   `//tensorflow/contrib/distributions:operator_pd_vdvt_update_test`  

   

##Step 4: Clean up (Optional)  

    ```shell
    sudo rm /<source_root>/gcc-5.4.0.tar.gz
    sudo rm /<source_root>/apache-maven-3.3.9-bin.tar.gz
    ```  

## References:
   https://www.tensorflow.org/  
   http://bazel.io/  
   https://github.com/google/protobuf/  
   https://github.com/tensorflow/tensorflow  
   https://github.com/grpc/grpc-java  
   https://github.com/bazelbuild/bazel  
