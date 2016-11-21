The instructions provided below specify the steps to build [MongoDB](http://mongodb.org/) 3.0.4 on Linux on the IBM z Systems for RHEL 7.1/7.2, SLES 12/12-SP1 and Ubuntu 16.04.

## Building MongoDB

 _**General Notes:**_  
_* When following the steps below please use a standard permission user unless otherwise specified._

_* A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

1. Building MongoDB requires a lot of disk space. Before attempting the build, ensure that you have 30GB of free space in the file system. You will also need root access to perform some of the steps below, e.g. installing libraries into system locations.

2. MongoDB depends on the V8 JavaScript engine. It includes V8 3.12 in its source tree, but that version does not support z Systems, so it will need to be linked with [a version of V8 that has been ported to z](https://github.com/andrewlow/v8z/). Fetch the V8z code from the 3.14-s390 branch, build the 64-bit shared library (libv8.so), and install it into the normal system location, usually under /usr/lib64/. See this article for complete instructions on building V8 3.14: [Building V8 libraries](https://github.com/ibm-linux-on-z/docs/wiki/Building-V8-libraries).

3. Building MongoDB requires SCons. On RHEL 7.7/7.2, download scons-2.3.4-1.noarch.rpm from [SCons project page](http://prdownloads.sourceforge.net/scons/scons-2.3.4-1.noarch.rpm) and install it like this:

        sudo rpm -i scons-2.3.4-1.noarch.rpm

   When building on SLES 12/12-SP1, SCons is included in the SLES 12 SDK, and can be installed from the DVD or online repository like this:

        sudo zypper install scons
  
  	When building on Ubuntu 16.04, SCons can be installed from the online repository like this:
  
  		sudo apt-get install scons
        
  	When building on Ubuntu 16.04, install g++ using the following steps
    	
		sudo apt-get install -y software-properties-common
        sudo add-apt-repository ppa:ubuntu-toolchain-r/test
        sudo apt-get update; sudo apt-get install gcc-4.8 g++-4.8
        sudo update-alternatives --remove-all gcc 
		sudo update-alternatives --remove-all g++
		sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 20
		sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.8 20
		sudo update-alternatives --config gcc
		sudo update-alternatives --config g++

4. Clone the latest MongoDB 3.0 branch from our GitHub repo, which includes the changes for big-endian platforms.
		
		cd /<source_root>/
        git clone https://github.com/linux-on-ibm-z/mongo mongo
        cd mongo
        git checkout v3.0-s390

5. In order to use the V8 APIs in libv8.so, MongoDB requires the corresponding V8 headers. If you did not install the V8 header files in a system location (e.g. /usr/include/ or /usr/local/include/), copy them from the v8z/include directory into mongo/src/.
		
		cd /<source_root>/
        cp v8z/include/*.h mongo/src/

6. In the mongo/ directory, execute the following command to build MongoDB:
		
		cd /<source_root>/mongo
        sudo scons --opt --use-system-v8 --allocator=system all

## Building MongoDB tools

1. In version 3.0, tools such as mongodump, mongoexport, mongoimport and mongorestore have been rewritten in Go and moved to a different GitHub repository. Clone the source code and check out release 3.0.4:
		
		cd /<source_root>/
        git clone https://github.com/mongodb/mongo-tools
        cd mongo-tools
        git checkout r3.0.4

2. Install go according to the instructions in [Building go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7).( For RHEL 7.1/7.2 and SLES 12/12-SP1 )

	For Ubuntu 16.04:
	
		sudo apt-get install gccgo golang

3. Make sure the gccgo executables are in your search path:( For RHEL 7.1/7.2 and SLES 12/12-SP1 )

        export PATH=/opt/gccgo/bin:$PATH
        export LD_LIBRARY_PATH=/opt/gccgo/lib64:$LD_LIBRARY_PATH

4. Edit the script `/mongo-tools/build.sh` build command on line 22:( For RHEL 7.1/7.2 and SLES 12/12-SP1 )

    <pre>
go build -o "bin/$i" -ldflags "-X github.com/mongodb/mongo-tools/common/options.Gitspec `git rev-parse HEAD`" -tags "$tags" "$i/main/$i.go"
</pre>
    
	Change the above line to:
	
	<pre>
go build -o "bin/$i" -ldflags "-X github.com/mongodb/mongo-tools/common/options.Gitspec=`git rev-parse HEAD`" -tags "$tags" "$i/main/$i.go"
</pre>

	
5. Run the script to build MongoDB tools:

        ./build.sh

## Testing (Optional)

1. To run self-verifying tests, [PyMongo](http://api.mongodb.org/python/current/) must be installed. To install PyMongo, build the driver from source:

		cd /<source_root>/
        git clone git://github.com/mongodb/mongo-python-driver.git pymongo
        cd pymongo
        sudo python setup.py install

2.	Copy the libv8.so file to /usr/lib and add it to the library path.

		export LD_LIBRARY_PATH=/usr/lib:LD_LIBRARY_PATH
		sudo cp -v /usr/local/lib64/libv8.so /usr/lib/
		sudo /sbin/ldconfig -v

3. To run the C++ unit tests, re-run the build command in the MongoDB build directory, but replace the target `all` with `smokeCppUnittests`:

        cd /<source_root>/mongo
        sudo scons --opt --use-system-v8 --allocator=system smokeCppUnittests
              
   To run the server smoke tests, you must copy all the MongoDB tools into the MongoDB server build directory, e.g.

        cd /<source_root>/mongo
        cp ../mongo-tools/bin/* .

   Then you can run the server smoke tests by re-running the build command with `--smokedbprefix=/tmp smoke`:

        sudo scons --opt --use-system-v8 --allocator=system --smokedbprefix=/tmp smoke

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