We have successfully built [V8z](https://github.com/andrewlow/v8z/) and [MongoDB](http://mongodb.org/) 2.4.9 with GCC 4.4.7. We have also built MongoDB 2.6.6 with GCC 4.8.2. Here is how we did it.

1. Build the 31-bit and 64-bit libv8.so, and install them into the normal system locations, usually under /usr/lib or /usr/lib64. See this article for instructions on building the library: [Building V8 libraries](https://github.com/ibm-linux-on-z/docs/wiki/Building-V8-libraries).

2. Clone the latest MongoDB fork for Power, which includes the changes for big-endian platforms.

        git clone https://github.com/ibmsoe/mongo
        cd mongo

3. As of March 2015, the work on the default v2.6.6-ppc branch is almost complete, and can be used for evaluation purposes. If you want to go back to the more stable 2.4.9 branch, execute these commands:

        git fetch
        git checkout r2.4.9-ppc

4. Building MongoDB requires SCons. Download scons-2.3.1-1.noarch.rpm from [SCons 2.3.1 Downloads](http://sourceforge.net/projects/scons/files/scons/2.3.1) and install it like this:

        rpm -i scons-2.3.1-1.noarch.rpm

5. If you did not install the V8 header files in a system location (e.g. /usr/include or /usr/local/include), copy them from the v8z/include directory into mongo/src. In order to use the V8 APIs in libv8.so, mongo requires the corresponding V8 headers.

6. In the mongo directory, execute the following to build:

  (31-bit)

      scons --31 --usev8 --use-system-v8 all

  (64-bit)

      scons --64 --usev8 --use-system-v8 all

  You can replace "all" with specific targets you are interested in building, e.g. mongo, mongod, etc.

7. The binaries will be output in the mongo directory. To install them properly, execute these commands:

        for i in bsondump mongo mongobridge mongod mongodump \
                 mongoexport mongofiles mongoimport mongooplog \
                 mongoperf mongorestore mongos mongostat mongotop ; do
            strip $i
            cp $i /usr/local/bin
            chmod 755 /usr/local/bin/$i
        done