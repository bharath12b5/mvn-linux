# Building HBase

HBase 1.2.4 has been built successfully on Linux on z Systems for Ubuntu 16.04 by following these instructions.

_**General Notes:**_  

* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Prerequisites(For IBM SDK only):

  * HBase needs Hadoop jar while building. To build HBase with IBM sdk, Hadoop (2.5.1) should be built first with IBM sdk. If it not compiled then while starting the HBase shell, it will throw an error message saying `Unable to find JAAS classes:com.ibm.security.auth.LinuxPrincipal`

### Step 1: Install the dependencies 

 * Ubuntu 16.04
    
 _**Note:** Set JAVA_HOME to your defined java path. Also set JAVA_HOME/bin to PATH variable properly._
	
	*  With IBM SDK   

	   Download IBM Java 8 sdk binary from [IBM SDK 8](https://developer.ibm.com/javasdk/downloads/) and follow the instructions as per given in the link.  

			sudo apt-get update
			sudo apt-get install -y git wget maven tar make gcc ant

	*  With Open JDK
		
			sudo apt-get update
			sudo apt-get install -y git openjdk-8-jdk wget maven tar make gcc ant
			

### Step 2: Set Environment Variable( for IBM SDK only)
   
   To start HBase shell and there are some test cases which require `HADOOP_USER_NAME` environment variable set 

   ```
      export HADOOP_USER_NAME="hadoop"
   ```

### Step 3: Build and install HBase
  1. Get the source
     ```
        cd /<source_root>/
        git clone https://github.com/apache/hbase.git
        cd hbase
        git checkout rel/1.2.4
     ```

  2. Build HBase

      ```
         mvn package -DskipTests
      ```

  4. Run test cases(Optional)  

      ```
         mvn test -fn
      ```

       _**Note:**_ 

       * Following test failures are observed on s390x and investigation is in progress

          * org.apache.hadoop.hbase.filter.TestFuzzyRowFilter (fails consistently with openjdk)
          * org.apache.hadoop.hbase.thrift.TestThriftHttpServer (fails intermittently with openjdk and IBM sdk)

### Step 4: Start HBase shell

  1. HBase needs a native library (libjffi-1.0.so: java foreign language interface). Get `jffi` source code and build with `ant`

     ```
        cd /<source_root>/
        wget https://github.com/jnr/jffi/archive/1.0.0.tar.gz
        tar -xvf 1.0.0.tar.gz
        cd jffi-1.0.0/
     ```

  2. Edit `jni/GNUmakefile` file 

      ```diff
         CFLAGS += -mwin32 -D_JNI_IMPLEMENTATION_
         LDFLAGS += -Wl,--add-stdcall-alias
         PICFLAGS=
-    SOFLAGS += -shared -mimpure-text -static-libgcc
+    SOFLAGS += -shared -static-libgcc
         PREFIX =
         JNIEXT=dll
         AR = x86_64-w64-mingw32-ar
      ```


      ```diff
         endif
         LDFLAGS += -Wl,--add-stdcall-alias $(CRTMT.O)
         PICFLAGS=
-    SOFLAGS += -shared -mimpure-text -static-libgcc
+    SOFLAGS += -shared -static-libgcc
         PREFIX =
         JNIEXT=dll
       endif

      ```

      ```diff
       ifeq ($(OS), linux)
-    SOFLAGS = -shared -mimpure-text -static-libgcc -Wl,-soname,$(@F) -Wl,-O1
+    SOFLAGS = -shared -static-libgcc -Wl,-soname,$(@F) -Wl,-O1
         CFLAGS += -pthread
       endif

      ```

      ```diff
         CFLAGS += -D__EXTENSIONS__ -std=c99
         LD = /usr/ccs/bin/ld
-    SOFLAGS = -shared -static-libgcc -mimpure-text
+    SOFLAGS = -shared -static-libgcc
         LIBS += -ldl
         STRIP = strip

      ```

  3. Edit `libtest/GNUmakefile` file 

      ```diff
         OFLAGS = -O2 $(JFLAGS)
         WFLAGS = -W -Werror -Wall -Wno-unused -Wno-parentheses
         PICFLAGS = -fPIC
-    SOFLAGS = -shared -mimpure-text -Wl,-O1
+    SOFLAGS = -shared -Wl,-O1
         LDFLAGS += $(SOFLAGS)

      ```

  4. Build jffi 

     ```
        ant
     ```
     _**Note:** There are some test case failures. The library file is created in `/<source_root>/jffi-1.0.0/build/jni/libjffi-1.0.so`_ 

  5. Modify `jruby-complete-1.6.8.jar` 

     ```
        mkdir /<source_root>/jar_tmp
        cp ~/.m2/repository/org/jruby/jruby-complete/1.6.8/jruby-complete-1.6.8.jar /<source_root>/jar_tmp
        cd /<source_root>/jar_tmp
        jar xf jruby-complete-1.6.8.jar
        mkdir jni/s390x-Linux
        cp /<source_root>/jffi-1.0.0/build/jni/libjffi-1.0.so jni/s390x-Linux
        jar uf jruby-complete-1.6.8.jar jni/s390x-Linux/libjffi-1.0.so
        mv jruby-complete-1.6.8.jar ~/.m2/repository/org/jruby/jruby-complete/1.6.8/jruby-complete-1.6.8.jar
     ```


  6. Start HBase shell
     ```
        cd /<source_root>/hbase
        bin/start-hbase.sh
        bin/hbase shell
     ```

###Reference:
https://hbase.apache.org/

https://github.com/apache/hbase

