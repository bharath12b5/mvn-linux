The V8 JavaScript engine is very popular and is included by many different projects. It can be built as a set of static libraries, or a single shared library. Many projects prefer linking V8 statically, while others might prefer linking multiple programs against a shared library, in the interest of footprint. The following instructions should produce both types of libraries on RHEL 7 and SLES 12.

Start by installing the prerequisites:

(RHEL)

    sudo yum install subversion

(SLES)

    sudo zypper install subversion

Clone depot_tools from the Chromium project, which contains a number of tools needed to build V8:

    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
    export PATH=`pwd`/depot_tools:$PATH


Then clone the V8z code from GitHub, and check out the current stable release 3.28:

    git clone https://github.com/andrewlow/v8z.git
    cd v8z
    git checkout 3.28-s390

Note that V8 3.28 or newer will require Python 2.7, which is not available from RHEL 6 or SLES 11 SP3. You could try [[building Python 2.7.9]] first; alternatively, you can try using the 3.14 release instead. V8 3.28 should build correctly on RHEL 7 and SLES 12.

If you need to use the older stable branch 3.14 (e.g. for building [MongoDB](../Building MongoDB)), issue the command `git checkout 3.14-s390`.

Now run `make dependencies` to fetch the source code for all dependent libraries.

If you are building V8 3.14 on SLES 11 SP3, issue this command after running `make dependencies`:

    sed -i "s/crsT/crs/g" build/gyp/pylib/gyp/generator/make.py

This will prevent the `ar` command from failing when creating libraries during the build process.

### Building the static libraries

Execute:

    make s390x -j4

The 64-bit static libraries will be found as _lib*.a_ in the directory v8z/out/s390x.release/obj.target/tools/gyp/. Building V8 3.14 creates the following libraries:

    libpreparser_lib.a
    libv8_base.a
    libv8_nosnapshot.a
    libv8_snapshot.a

Building V8 3.28 creates these libraries:

    libv8_base.a
    libv8_libbase.a
    libv8_libplatform.a
    libv8_nosnapshot.a
    libv8_snapshot.a

The V8 3.28 build also generates other libraries, named libicuuc.a, libicudata.a and libicui18n.a, which are located in v8z/out/s390x.release/obj.target/third_party/icu/.

To build the 31-bit versions, just replace the word "s390x" in the above command with "s390". The 31-bit static libraries will be found in v8z/out/s390.release/obj.target/tools/gyp/ (and v8z/out/s390.release/obj.target/third_party/icu/, when building V8 3.28).

### Building the shared library

Execute:

    make s390x -j4 library=shared

The 64-bit shared library will be output as v8z/out/s390x.release/lib.target/libv8.so.

To build the 31-bit version, just replace the word "s390x" in the above command with "s390". The 31-bit shared library will be output as v8z/out/s390.release/lib.target/libv8.so.

When building V8 3.28, libicui18n.so and libicuuc.so are also created in the same directory as libv8.so.

### Header files

V8 header files will be required to build products invoking the V8 APIs. These header files are found in v8z/include/.

### Installing the V8 libraries and header files

The following script can help install all the artifacts to the correct locations on a Linux system (root access required):

    cd v8z
    cp -R include/* /usr/include/
    chmod 644 /usr/include/v8*h /usr/include/libplatform/libplatform.h
    
    cp out/s390x.release/lib.target/lib*.so /usr/lib64/
    chmod 755 /usr/lib64/libv8.so /usr/lib64/libicu*.so
    
    cp out/s390.release/lib.target/lib*.so /usr/lib/
    chmod 755 /usr/lib/libv8.so /usr/lib/libicu*.so
    
    cp out/s390x.release/obj.target/tools/gyp/lib*.a /usr/lib64/
    cp out/s390x.release/obj.target/third_party/icu/lib*.a /usr/lib64/
    chmod 644 /usr/lib64/libv8*.a /usr/lib64/libpreparser_lib.a /usr/lib64/libicu*.a
    
    cp out/s390.release/obj.target/tools/gyp/lib*.a /usr/lib/
    cp out/s390.release/obj.target/third_party/icu/lib*.a /usr/lib64/
    chmod 644 /usr/lib/libv8*.a /usr/lib/libpreparser_lib.a /usr/lib/libicu*.a
