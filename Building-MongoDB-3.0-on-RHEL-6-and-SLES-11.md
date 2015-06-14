[MongoDB](http://mongodb.org/) 3.0.4 has been ported to Linux on IBM z Systems. The following build instructions have been tested on RHEL 6 and SLES 11 SP3.

## Building MongoDB

1. MongoDB includes V8 3.12 in its source tree, but that version does not support z Systems, so it will need to be linked with [a version of V8 that has been ported to z](https://github.com/andrewlow/v8z/). Fetch the V8z code from the 3.14-s390 branch, build the 31-bit and 64-bit libv8.so, and install them into the normal system locations, usually under /usr/lib/ or /usr/lib64/. See this article for complete instructions: [Building V8 libraries](https://github.com/ibm-linux-on-z/docs/wiki/Building-V8-libraries).

2. Building MongoDB requires SCons. Download scons-2.3.1-1.noarch.rpm from [SCons 2.3.1 Downloads](http://sourceforge.net/projects/scons/files/scons/2.3.1) and install it like this:

        rpm -i scons-2.3.1-1.noarch.rpm

3. Building MongoDB 3.0 requires GCC 4.8.2 or newer. However, RHEL 6 and SLES 11 ship older versions of GCC, so it is necessary to build and install a newer GCC first. Download the [GCC 5.1.0 source code](ftp://gcc.gnu.org/pub/gcc/releases/gcc-5.1.0/gcc-5.1.0.tar.bz2) and issue the following commands:

        tar xjf gcc-5.1.0.tar.bz2
        cd gcc-5.1.0
        ./contrib/download_prerequisites
        mkdir build
        cd build
        ../configure --prefix=/opt/gcc-5.1.0 \
            --enable-languages="c,c++,go" --enable-shared --with-system-zlib \
            --enable-threads=posix --enable-__cxa_atexit --enable-checking \
            --enable-gnu-indirect-function --disable-multilib --disable-bootstrap
        make
        make install

   Note that it is necessary to build the Go compiler in GCC as well, as it is needed for the MongoDB tools (see below).

4. Clone the latest MongoDB 3.0 fork, which includes the changes for big-endian platforms.

        git clone https://github.com/linux-on-ibm-z/mongo mongo
        cd mongo
        git checkout v3.0-s390

5. In order to use the V8 APIs in libv8.so, MongoDB requires the corresponding V8 headers. If you did not install the V8 header files in a system location (e.g. /usr/include/ or /usr/local/include/), copy them from the v8z/include directory into mongo/src/.

        cp v8z/include/*.h mongo/src/

6. In the mongo/ directory, execute the following command to build MongoDB:

        scons --cc=/opt/gcc-5.1.0/bin/gcc --cxx=/opt/gcc-5.1.0/bin/g++ \
              --static-libstdc++ --disable-warnings-as-errors --opt \
              --use-system-v8 --allocator=system --variant-dir=z all

   You can replace "all" with specific targets you are interested in building, e.g. mongo, mongod, etc.

## Building MongoDB tools

1. In version 3.0, tools such as mongodump, mongoexport, mongoimport and mongorestore have been rewritten in Go and moved to a different GitHub repository. Clone the source code and check out release 3.0.4:

        git clone https://github.com/mongodb/mongo-tools
        cd mongo-tools
        git checkout r3.0.4

2. Make sure the gccgo executables are in your search path:

        export PATH=/opt/gcc-5.1.0/bin:$PATH
        export LD_LIBRARY_PATH=/opt/gcc-5.1.0/lib64:$LD_LIBRARY_PATH

3. Edit the script build.sh to insert a gccgo flag into the `go build` command on line 22:

    <pre>
go build -o "bin/$i" <b><i>-gccgoflags '-static-libgo'</i></b> -ldflags ...
</pre>

4. Run the script to build MongoDB tools:

        ./build.sh

## Installation

The binaries will be output in the mongo/ and mongo-tools/bin/ directories. To install them properly, execute these commands:

    for i in mongo mongobridge mongod mongoperf mongos ; do
        cp mongo/$i /usr/local/bin/
        chmod 755 /usr/local/bin/$i
    done
    
    for i in bsondump mongodump mongoexport mongofiles mongoimport \
            mongooplog mongorestore mongostat mongotop ; do
        cp mongo-tools/bin/$i /usr/local/bin/
        chmod 755 /usr/local/bin/$i
    done