[MongoDB](http://mongodb.org/) 3.2.0 has been ported to Linux on IBM z Systems. The following build instructions have been tested on RHEL 7.1/7.2 and SLES 12/12-SP1.

## Building MongoDB

1. Building MongoDB requires a lot of disk space. Before attempting the build, ensure that you have 30GB of free space. You will also need root access to perform some of the steps below, e.g. installing libraries into system locations.

2. Building MongoDB requires SCons. On RHEL 7.1/7.2, download scons-2.3.4-1.noarch.rpm from [SCons project page](http://prdownloads.sourceforge.net/scons/scons-2.3.4-1.noarch.rpm) and install it like this:

        rpm -i scons-2.3.4-1.noarch.rpm

   When building on SLES 12, SCons is included in the SLES 12 SDK, and can be installed from the DVD or online repository like this:

        sudo zypper install scons

3. Clone the latest MongoDB 3.2 branch from our GitHub repo, which includes the changes for big-endian platforms.

        git clone https://github.com/linux-on-ibm-z/mongo mongo
        cd mongo
        git checkout v3.2-s390

4. In the mongo/ directory, create a file named version.json with the following content:

   ```json
   { "version": "3.2.0", "githash": "b80311d7fcfaae95c434ade551949921925234e6" }
   ```

   This helps prevent a unit test from failing due to the introduction of [this upstream change](https://github.com/linux-on-ibm-z/mongo/commit/457246ef9b013b30cafa4bb45125dc2e4193d6c2).

5. In the mongo/ directory, execute the following command to build MongoDB:

        scons --opt --allocator=system all

## Building MongoDB tools

1. Since version 3.0, tools such as mongodump, mongoexport, mongoimport and mongorestore have been rewritten in Go and moved to a different GitHub repository. Clone the source code and check out release master-branch:

        git clone https://github.com/mongodb/mongo-tools
        cd mongo-tools
        git checkout r3.2.0

2. Install gccgo according to the instructions in [[Building gccgo]].

3. Make sure the gccgo executables are in your search path:

        export PATH=/opt/gccgo/bin:$PATH
        export LD_LIBRARY_PATH=/opt/gccgo/lib64:$LD_LIBRARY_PATH

   (This will change your default GCC version to 6.0. If you need to build MongoDB again, the PATH variable needs to be changed back to pick up GCC 4.8.x to before re-building MongoDB.)

4. Edit the script mongo-tools/build.sh to insert a gccgo flag into the `go build` command on line 22:

    <pre>
go build -o "bin/$i" <b><i>-gccgoflags '-static-libgo'</i></b> -ldflags ...
</pre>

5. Run the script to build MongoDB tools:

        cd mongo-tools
        ./build.sh

## Testing (Optional)

1. To run self-verifying tests, two Python modules [PyMongo](http://api.mongodb.org/python/current/) and [PyYAML](http://pyyaml.org/wiki/PyYAML) must be installed.

   To check if you have those two modules installed, run:

        python -c "import pymongo; import yaml"

   If both modules are installed already, nothing will be printed to the console. Otherwise, the console will print an error message, such as "No module named yaml" or "No module named pymongo".

   To install PyMongo on RHEL 7.1/7.2 or SLES 12/12-SP1, build the driver from source:

        git clone git://github.com/mongodb/mongo-python-driver.git pymongo
        cd pymongo
        python setup.py install

   PyYAML should be included in RHEL 7.1/7.2 by default. In case you do not have PyYAML installed, install it with YUM:

        sudo yum install PyYAML

   To install PyYAML on SLES 12/12-SP1, build it from source:

        git clone https://github.com/yaml/pyyaml.git
        cd pyyaml
        python setup.py install

2. There are 3 main categories of tests:

   * JavaScript integration tests
   * "New-style" C++ unit tests
   * "Old-style" (deprecating) C++ dbtests

   To run JavaScript integeration tests:

        cd mongo
        python buildscripts/resmoke.py --suites=core

   To run C++ unit tests:

        cd mongo
        python buildscripts/resmoke.py --suites=unittests

   To run C++ dbtests:

        cd mongo
        ./dbtest

   More details about testing can be found at [MongoDB Manual](https://docs.mongodb.org/manual/contributors/tutorial/test-the-mongodb-server/).

## Installation

The binaries will be output in the mongo/ and mongo-tools/bin/ directories. To install them properly, execute these commands:

    for i in mongo mongobridge mongod mongoperf mongos ; do
        cp mongo/$i /usr/local/bin/
        chmod 755 /usr/local/bin/$i
    done

    for i in mongodump mongoexport mongofiles mongoimport \
            mongooplog mongorestore mongostat mongotop ; do
        cp mongo-tools/bin/$i /usr/local/bin/
        chmod 755 /usr/local/bin/$i
    done

## References

- [MongoDB 3.2 manual](http://docs.mongodb.org/manual/)
- [More information on running MongoDB for the first time](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-linux/#run-mongodb)
