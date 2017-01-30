<!---PACKAGE:rethinkDB--->
<!---DISTRO:SLES 12 SP2--->

# Building RethinkDB v2.3.5

[rethinkDB](https://www.rethinkdb.com/) RethinkDB is the open-source, scalable database that makes building realtime apps dramatically easier. The stable release of RethinkDB 2.3.5 has been built and tested on Linux on z Systems.  The following instructions can be used for SLES 12 SP2.

### _**General Note:**_
* _When following the steps below please use a standard permission user unless otherwise specified_
* _A directory \<source root\> will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it_

## Building and Installing RethinkDB v2.3.5

### Step 1: Install the Dependencies

    ```sudo zypper in make gcc gcc-c++ protobuf-devel ncurses-devel boost-devel tar wget m4 which openssl-devel libcurl-devel```
   
### Step 2: Download and build RethinkDB
```
wget https://download.rethinkdb.com/dist/rethinkdb-2.3.5.tgz
tar xf rethinkdb-2.3.5.tgz
cd rethinkdb-2.3.5
```

Modify following files to enable s390x support
```
vi src/rpc/connectivity/cluster.cc
#if defined (__x86_64__) || defined (_WIN64) || defined(__s390x__)
vi src/arch/runtime/context_switching.cc
#ifndef _WIN32
```
Use 3.28 V8 packages for s390x and modify v8.sh as follows
```
vi ./mk/support/pkg/v8.sh
cd $build_dir
rm -rf $build_dir/v8z
rm -rf $build_dir/depot_tools
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH=$(pwd)/depot_tools:$PATH
git clone https://github.com/ibmruntimes/v8z
cd v8z
git checkout 3.28-s390
make dependencies
make s390x -j4
make s390x -j4 library=shared
cp -vR include/* /usr/include/
chmod 644 /usr/include/libplatform/libplatform.h
chmod 644 /usr/include/v8*h
cp -v out/s390x.release/lib.target/lib*.so /usr/local/lib64/
chmod -f 755 /usr/local/lib64/libv8.so
cp -v out/s390x.release/obj.target/tools/gyp/lib*.a /usr/local/lib64/
cp -v out/s390x.release/obj.target/third_party/icu/lib*.a /usr/local/lib64/
chmod -f 755 /usr/local/lib64/libicu*.so
chmod -f 644 /usr/local/lib64/libv8*.a /usr/local/lib64/libicu*.a
# pkg_make $arch.$mode CXX=$CXX LINK=$CXX LINK.target=$CXX GYPFLAGS="-Dwerror= $arch_gypflags" V=1
    echo $build_dir
    for lib in `find "$build_dir/v8z/out/s390x.release" -maxdepth 1 -name \*.a` `find "$build_dir/v8z/out/s390x.release/obj.target" -name \*.a`; do
        name=`basename $lib`
        cp $lib "$install_dir/lib/${name/.$arch/}"
    done
    touch "$install_dir/lib/libv8.a" # Create a dummy libv8.a because the makefile looks for it

```
Configure and make RethinkDB
```
./configure --allow-fetch --dynamic jemalloc
make
```