The V8 JavaScript engine is very popular and is included by many different projects. It can be built as a set of static libraries, or a single shared library. Many projects prefer linking V8 statically, while others might prefer linking multiple programs against a shared library, in the interest of footprint. The following instructions should produce both types of libraries on all Linux distributions.

Start by cloning the V8z code from GitHub, and using the current stable release 3.28:

    git clone https://github.com/andrewlow/v8z.git
    cd v8z
    git checkout 3.28-s390
    make dependencies

If you need to use the older stable branch 3.14 (e.g. for building [MongoDB](../Building MongoDB)), use the command `git checkout 3.14-s390` instead.

If you are building V8z on SLES 11 SP3, issue this command after running "make dependencies":

    sed -i "s/crsT/crs/g" build/gyp/pylib/gyp/generator/make.py

This will prevent the `ar` command from failing when creating libraries during the build process.

### Building the static libraries

Execute:

    make s390x -j4

The 64-bit static libraries will be found as _libpreparser_lib.a_, _libv8_base.a_, _libv8_nosnapshot.a_, and _libv8_snapshot.a_ in the directory _v8z/out/s390x.release/obj.target/tools/gyp/_.

To build the 31-bit versions, just replace the word "s390x" in the above command with "s390". The 31-bit static libraries will be found in _v8z/out/s390.release/obj.target/tools/gyp/_.

### Building the shared library

Execute:

    make s390x -j4 library=shared

The 64-bit shared library will be output as _v8z/out/s390x.release/lib.target/libv8.so_.

To build the 31-bit version, just replace the word "s390x" in the above command with "s390". The 31-bit shared library will be output as _v8z/out/s390.release/lib.target/libv8.so_.

### Header files

V8 header files will be required to build products invoking the V8 APIs. These header files are found in _v8z/include/_.

### Installing the V8 libraries and header files

The following script can help installing all the artifacts to the correct locations on a Linux system (root access required):

    cd v8z
    cp include/* /usr/include/
    chmod 644 /usr/include/v8*h
    cp out/s390x.release/lib.target/libv8.so /usr/lib64/
    chmod 755 /usr/lib64/libv8.so
    cp out/s390.release/lib.target/libv8.so /usr/lib/
    chmod 755 /usr/lib/libv8.so
    cp out/s390x.release/obj.target/tools/gyp/lib*.a /usr/lib64/
    chmod 644 /usr/lib64/libv8*.a /usr/lib64/libpreparser_lib.a
    cp out/s390.release/obj.target/tools/gyp/lib*.a /usr/lib/
    chmod 644 /usr/lib/libv8*.a /usr/lib/libpreparser_lib.a
