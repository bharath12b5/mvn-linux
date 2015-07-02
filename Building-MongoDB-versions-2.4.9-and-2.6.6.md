[MongoDB](http://mongodb.org/) can be built for a Linux on z System running RHEL 7.1/6.6 and SLES 12/11 by following the instructions below.
Note that [V8z](https://github.com/andrewlow/v8z/) will also need to be built, as referenced below.
These instructions have been used to successfully build versions 2.4.9 and 2.6.6 of MongoDB.

_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified. It's suggested that you create a new working directory from which to execute the instructions below (unless otherwise specified)._


1. Install the build time dependencies

    On RHEL systems:

        sudo yum install git make git-svn gcc-c++ wget

    On SLES systems:

        sudo zypper install git make git-svn gcc-c++ wget

  You may already have some of these packages installed - just install any that are missing.
  
2. MongoDB includes V8 3.12 in its source tree, but that version does not support IBM z Systems, so it will need to be linked with a version of V8 that has been ported to z. Fetch the V8z code from the 3.14-s390 branch, build the 64-bit libv8.so, and install it into the normal system locations, usually under /usr/lib/ or /usr/lib64/. See this article for complete instructions: [Building V8 libraries](https://github.com/linux-on-ibm-z/docs/wiki/Building-V8-libraries).

3. Building MongoDB requires SCons. Download scons-2.3.1-1.noarch.rpm from [SCons 2.3.1 Downloads](http://sourceforge.net/projects/scons/files/scons/2.3.1) and install it like this:

        wget http://sourceforge.net/projects/scons/files/scons/2.3.1/scons-2.3.1-1.noarch.rpm
        sudo rpm -i scons-2.3.1-1.noarch.rpm

4. Clone the latest MongoDB fork for Power, which includes the changes for big-endian platforms.  Then change into the mongo directory.

        git clone https://github.com/ibmsoe/mongo mongo
        cd mongo

5. As of March 2015, the work on the default v2.6.6-ppc branch is almost complete, and can be used for evaluation purposes. If you want to go back to the more stable 2.4.9 branch, execute these commands in the mongo/ directory:

        git fetch
        git checkout r2.4.9-ppc

6. In order to use the V8 APIs in libv8.so, MongoDB requires the corresponding V8 headers. If you did not install the V8 header files in a system location (e.g. /usr/include/ or /usr/local/include/), copy them from the v8z/include directory into mongo/src/.

        cp <v8z-path>/include/*.h src/

7. In the mongo/ directory, execute the following to build:

  (64-bit)

      scons --64 --usev8 --use-system-v8 all

  You can replace "all" with specific targets you are interested in building, e.g. mongo, mongod, etc.

8. The binaries will be output in the mongo directory. To install them properly, execute these commands:

  ```bash
  for i in bsondump mongo mongobridge mongod mongodump \
           mongoexport mongofiles mongoimport mongooplog \
           mongoperf mongorestore mongos mongostat mongotop ; do
      strip $i
      sudo cp $i /usr/local/bin/
      sudo chmod 755 /usr/local/bin/$i
  done
  ```

9. Once the binaries have been installed, the 'build' subdirectory may be removed, in case you need to recover disk space.


General purpose smoke tests are available from [MongoDB Tests](http://www.mongodb.org/about/contributors/tutorial/test-the-mongodb-server/) and may be used to validate the MongoDB build as follows:

1. Clone a recent version of pymongo

        git clone git://github.com/mongodb/mongo-python-driver.git pymongo
        cd pymongo

    As of May 2015, the latest version of pymongo for running the smoke tests was found to be v2.7 (for RHEL 7 and RHEL 6) and v2.6 (for SLES 12 and SLES 11). To revert to the relevant version e.g. for SLES 12 :
    
        git checkout v2.6

2. Install pymongo by running the following command (from the pymongo directory)

        sudo python setup.py install

3. Run the general purpose smoke tests

    Before running tests for the first time, you will need to create the mongod data directory. By default, mongod writes data to the /data/db/ directory.  Ensure that the user that runs the tests has ownership of this directory and read/write permissions to it.

    **For MongoDB 2.6.6** the general purpose smoke tests are found in mongo/jstests/core. To run an individual test:

        cd mongo
        python buildscripts/smoke.py --mode=files jstests/core/all.js

    To run all tests it's suggested that you create and run a shell script such as the following:

  ```bash
  echo '#!/bin/bash' > run_smoke_tests.sh
  echo 'mkdir -p smoke_logs' >> run_smoke_tests.sh
  echo 'find jstests/core -name "*.js" | sort | \' >> run_smoke_tests.sh
  echo 'while read -r line ; do' >> run_smoke_tests.sh
  echo '    dname=`dirname $line`' >> run_smoke_tests.sh
  echo '    fname=`basename $line`' >> run_smoke_tests.sh
  echo '    echo `date` "$line" >> smoke_logs/smoke_list.log' >> run_smoke_tests.sh
  echo '    rm -r /data/db/*' >> run_smoke_tests.sh
  echo '    mkdir -p smoke_logs/$dname' >> run_smoke_tests.sh
  echo '    python buildscripts/smoke.py --mode=files $dname/$fname &> smoke_logs/$dname/$fname.log' >> run_smoke_tests.sh
  echo 'done' >> run_smoke_tests.sh
  chmod +x run_smoke_tests.sh
  ./run_smoke_tests.sh &
  ```

    The above script will write a log of each test into smoke_logs/jstests/core/ which will indicate whether the test succeeded. It assumes that you are using the default db directory /data/db which it clears between tests and it will log the start time for each test in smoke_logs/smoke_list.log.

    **For MongoDB 2.4.9** the general purpose smoke tests are found directly under mongo/jstests. To run an individual test:

        cd mongo
        python buildscripts/smoke.py --mode=files jstests/all.js

    To run all tests it's suggested that you create and run a shell script such as was done for MongoDB 2.6.6 above. For 2.4.9, the same script may be used except that the 'find' line above will need to be replaced with the following:
    
  ```bash
  echo 'find jstests -maxdepth 1 -name "*.js" | grep -v "jstests/_" | sort | \' >> run_smoke_tests.sh
  ```
   