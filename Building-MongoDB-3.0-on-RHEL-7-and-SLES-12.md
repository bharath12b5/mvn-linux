[MongoDB](http://mongodb.org/) 3.0.4 has been ported to Linux on IBM z Systems. The following build instructions have been tested on RHEL 7.1 and SLES 12.

## Building MongoDB

1. MongoDB includes V8 3.12 in its source tree, but that version does not support z Systems, so it will need to be linked with [a version of V8 that has been ported to z](https://github.com/andrewlow/v8z/). Fetch the V8z code from the 3.14-s390 branch, build the 64-bit shared library (libv8.so), and install it into the normal system location, usually under /usr/lib64/. See this article for complete instructions on building V8 3.14: [Building V8 libraries](https://github.com/ibm-linux-on-z/docs/wiki/Building-V8-libraries).

2. Building MongoDB requires SCons. On RHEL 7, download scons-2.3.4-1.noarch.rpm from [SCons project page](http://prdownloads.sourceforge.net/scons/scons-2.3.4-1.noarch.rpm) and install it like this:

        rpm -i scons-2.3.4-1.noarch.rpm

   When building on SLES 12, SCons is included in the SLES 12 SDK, and can be installed from the DVD or online repository like this:

        zypper install scons

5. Clone the latest MongoDB 3.0 branch from our GitHub repo, which includes the changes for big-endian platforms.

        git clone https://github.com/linux-on-ibm-z/mongo mongo
        cd mongo
        git checkout v3.0-s390

5. In order to use the V8 APIs in libv8.so, MongoDB requires the corresponding V8 headers. If you did not install the V8 header files in a system location (e.g. /usr/include/ or /usr/local/include/), copy them from the v8z/include directory into mongo/src/.

        cp v8z/include/*.h mongo/src/

6. In the mongo/ directory, execute the following command to build MongoDB:

        scons --opt --use-system-v8 --allocator=system all

## Testing (Optional)

1. To run self-verifying tests, [PyMongo](http://api.mongodb.org/python/current/) must be installed. To install PyMongo on RHEL 6 or SLES 11, build the driver from source:

        git clone git://github.com/mongodb/mongo-python-driver.git pymongo
        cd pymongo
        python setup.py install

2. To run the C++ unit tests, re-run the build command but replace the target `all` with `smokeCppUnittests`:

        scons --opt --use-system-v8 --allocator=system smokeCppUnittests
              
   To run the server smoke tests, re-run the build command with `--smokedbprefix=/tmp smoke`:

        scons --opt --use-system-v8 --allocator=system --smokedbprefix=/tmp smoke

## Building MongoDB tools

1. In version 3.0, tools such as mongodump, mongoexport, mongoimport and mongorestore have been rewritten in Go and moved to a different GitHub repository. Clone the source code and check out release 3.0.4:

        git clone https://github.com/mongodb/mongo-tools
        cd mongo-tools
        git checkout r3.0.4

2. Install gccgo according to the instructions in [[Building gccgo]].

3. Make sure the gccgo executables are in your search path:

        export PATH=/opt/gccgo/bin:$PATH
        export LD_LIBRARY_PATH=/opt/gccgo/lib64:$LD_LIBRARY_PATH

4. Edit the script mongo-tools/build.sh to insert a gccgo flag into the `go build` command on line 22:

    <pre>
go build -o "bin/$i" <b><i>-gccgoflags '-static-libgo'</i></b> -ldflags ...
</pre>

5. Run the script to build MongoDB tools:

        cd mongo-tools
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