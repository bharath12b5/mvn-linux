The instructions provided below specify the steps to build [MongoDB](http://mongodb.org/) 3.0.4 on Linux on the IBM z Systems for RHEL 6 and SLES 11 SP3.

## Building MongoDB

 _**General Notes:**_  
_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

1. Building MongoDB requires a lot of disk space. Before attempting the build, ensure that you have 30GB of free space in the file system. You will also need root access to perform some of the steps below, e.g. installing libraries into system locations.

2. MongoDB depends on the V8 JavaScript engine. It includes V8 3.12 in its source tree, but that version does not support z Systems, so it will need to be linked with [a version of V8 that has been ported to z](https://github.com/andrewlow/v8z/). Fetch the V8z code from the 3.14-s390 branch, build the 64-bit shared library (libv8.so), and install it into the normal system location, usually under /usr/lib64/. See this article for complete instructions on building V8 3.14: [Building V8 libraries](https://github.com/ibm-linux-on-z/docs/wiki/Building-V8-libraries).

3. Building MongoDB requires SCons. Download scons-2.3.1-1.noarch.rpm from [SCons 2.3.1 Downloads](http://sourceforge.net/projects/scons/files/scons/2.3.1) and install it like this:

        rpm -i scons-2.3.1-1.noarch.rpm
		
4. Install dependency 
		
		sudo zypper install zlib-devel
			
5. Building MongoDB 3.0 requires GCC 4.8.2 or newer. However, RHEL 6 and SLES 11 ship older versions of GCC, so it is necessary to build and install a newer GCC first. Download the [GCC 5.1.0 source code](ftp://gcc.gnu.org/pub/gcc/releases/gcc-5.1.0/gcc-5.1.0.tar.bz2) and issue the following commands:

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
        sudo make install

   Note that it is necessary to build the Go compiler in GCC as well, as it is needed for the MongoDB tools (see below).

6. Clone the latest MongoDB 3.0 branch from our GitHub repo, which includes the changes for big-endian platforms.
        
		cd /<source_root>/
        git clone https://github.com/linux-on-ibm-z/mongo mongo
        cd mongo
        git checkout v3.0-s390

7. In order to use the V8 APIs in libv8.so, MongoDB requires the corresponding V8 headers. If you did not install the V8 header files in a system location (e.g. /usr/include/ or /usr/local/include/), copy them from the v8z/include directory into mongo/src/.
        
		cd /<source_root>/
        cp v8z/include/*.h mongo/src/

8. In the mongo/ directory, execute the following command to build MongoDB:

		cd /<source_root>/mongo
        scons --cc=/opt/gcc-5.1.0/bin/gcc --cxx=/opt/gcc-5.1.0/bin/g++ \
              --static-libstdc++ --disable-warnings-as-errors --opt \
              --use-system-v8 --allocator=system --variant-dir=z all

## Building MongoDB tools

1. In version 3.0, tools such as mongodump, mongoexport, mongoimport and mongorestore have been rewritten in Go and moved to a different GitHub repository. Clone the source code and check out release 3.0.4:
        
		cd /<source_root>/
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

## Testing (Optional)

1. To run self-verifying tests, [PyMongo](http://api.mongodb.org/python/current/) must be installed. To install PyMongo, build the driver from source:
		
		cd /<source_root>/
        git clone git://github.com/mongodb/mongo-python-driver.git pymongo
        cd pymongo
        python setup.py install

2. To run the C++ unit tests, re-run the build command in the MongoDB build directory, but replace the target `all` with `smokeCppUnittests`:

        cd /<source_root>/mongo
        scons --cc=/opt/gcc-5.1.0/bin/gcc --cxx=/opt/gcc-5.1.0/bin/g++ \
              --static-libstdc++ --disable-warnings-as-errors --opt \
              --use-system-v8 --allocator=system --variant-dir=z smokeCppUnittests
              
   To run the server smoke tests, you must copy all the MongoDB tools into the MongoDB server build directory, e.g.

        cd /<source_root>/mongo
        cp /<source_root>/mongo-tools/bin/* .

   Then you can run the server smoke tests by re-running the build command with `--smokedbprefix=/tmp smoke`:

        scons --cc=/opt/gcc-5.1.0/bin/gcc --cxx=/opt/gcc-5.1.0/bin/g++ \
              --static-libstdc++ --disable-warnings-as-errors --opt \
              --use-system-v8 --allocator=system --variant-dir=z \
              --smokedbprefix=/tmp smoke

## Installation

The binaries will be output in the mongo/ and mongo-tools/bin/ directories. To install them properly, execute these commands:

    cd /<source_root>/
    for i in mongo mongobridge mongod mongoperf mongos ; do
        sudo cp mongo/$i /usr/local/bin/
        sudo chmod 755 /usr/local/bin/$i
    done
    
    for i in bsondump mongodump mongoexport mongofiles mongoimport \
            mongooplog mongorestore mongostat mongotop ; do
        sudo cp mongo-tools/bin/$i /usr/local/bin/
        sudo chmod 755 /usr/local/bin/$i
    done

## References

- [MongoDB 3.0 manual](http://docs.mongodb.org/manual/)
- [More information on running MongoDB for the first time](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-linux/#run-mongodb)