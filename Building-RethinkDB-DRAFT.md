<!---PACKAGE:RethinkDB--->
<!---DISTRO:Ubuntu 16.04--->

# Building RethinkDB v2.3.5

[rethinkDB](https://www.rethinkdb.com/) RethinkDB is the open-source, scalable database that makes building realtime apps dramatically easier. The stable release of RethinkDB 2.3.5 has been built and tested on Linux on z Systems.  The following instructions can be used for Ubuntu 16.04.

### _**General Note:**_
* _When following the steps below please use a standard permission user unless otherwise specified_
* _A directory \<source root\> will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it_

## Building and Installing RethinkDB v2.3.5

### Step 1: Install the Dependencies

```
sudo apt-get install build-essential protobuf-compiler python \
                     libprotobuf-dev libcurl4-openssl-dev \
                     libboost-all-dev libncurses5-dev \
                     libjemalloc-dev wget m4
```
    
### Step 2: Download and build RethinkDB
```
wget https://download.rethinkdb.com/dist/rethinkdb-2.3.5.tgz
tar xf rethinkdb-2.3.5.tgz
cd rethinkdb-2.3.5
```

##Modify following files to enable s390x support

Modify ./configure
```
    case "${MACHINE%%-*}" in
        s390x|x86_64|i?86)
```        
Modify ./src/rpc/connectivity/cluster.cc
```
#if defined (__x86_64__) || defined (_WIN64) || defined(__s390x__)
```
Modify ./src/arch/runtime/context_switching.cc
```
101 #elif defined(__arm__) || defined (__s390x__)
265 #if defined(__i386__) || defined(__x86_64__) || defined(__arm__) || defined (__s390x__)
```
Modify ./src/rdb_protocol/datum.cc and comment out these lines
```
//#ifndef BOOST_LITTLE_ENDIAN
//    static_assert(false, "This piece of code will break on big-endian systems.");
//#endif
//#ifndef BOOST_LITTLE_ENDIAN
//        static_assert(false, "This piece of code will break on little endian systems.");
//#endif
```
Modify ./src/rdb_protocol/terms/string.cc and change ICU function as follows
```
static bool is_combining_character(char32_t c) {
    //return (U_GET_GC_MASK(c) & U_GC_M_MASK) > 0;
    return (c!=' ');
}
static bool is_whitespace_character(char32_t c) {
    //return u_isUWhiteSpace(c);
    return std::isspace(c); //replace with non-ICU library
}

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
make s390x -j4 library=shared i18nsupport=off
cp -vR include/* /usr/include/
chmod 644 /usr/include/libplatform/libplatform.h
chmod 644 /usr/include/v8*h
cp -v out/s390x.release/lib.target/lib*.so /usr/local/lib/
chmod -f 755 /usr/local/lib/libv8.so
cp -v out/s390x.release/obj.target/tools/gyp/lib*.a /usr/local/lib/
chmod -f 644 /usr/local/lib/libv8*.a
/sbin/ldconfig -v
# pkg_make $arch.$mode CXX=$CXX LINK=$CXX LINK.target=$CXX GYPFLAGS="-Dwerror= $arch_gypflags" V=1
    for lib in `find "$build_dir/v8z/out/s390x.release" -maxdepth 1 -name \*.a` `find "$build_dir/v8z/out/s390x.release/obj.target" -name \*.a`; do
        name=`basename $lib`
        cp $lib "$install_dir/lib/${name/.$arch/}"
    done
    touch "$install_dir/lib/libv8.a" # Create a dummy libv8.a because the makefile looks for it

pkg_link-flags () {
    for lib in libv8_{base,libbase,snapshot,libplatform}; do
        echo "$install_dir/lib/$lib.a"
    done
```
Configure and make RethinkDB
```
./configure --allow-fetch --dynamic jemalloc
make
```

Currently ```make``` step is failing with following error:

```
root@lozlnxk:/localbox/rishi/rethinkdb-2.3.5# make
    [1/9] CC build/release/obj/extproc/js_job.o
    [2/9] LD build/release/rethinkdb
./build/release/obj/extproc/js_job.o: In function `v8::Value::ToString() const':
/localbox/rishi/rethinkdb-2.3.5/./build/external/v8_3.30.33.16/include/v8.h:6860: undefined reference to `v8::Value::ToString(v8::Isolate*) const'
./build/release/obj/extproc/js_job.o: In function `v8::Value::ToObject() const':
/localbox/rishi/rethinkdb-2.3.5/./build/external/v8_3.30.33.16/include/v8.h:6870: undefined reference to `v8::Value::ToObject(v8::Isolate*) const'
./build/release/obj/extproc/js_job.o: In function `v8::Value::ToString() const':
/localbox/rishi/rethinkdb-2.3.5/./build/external/v8_3.30.33.16/include/v8.h:6860: undefined reference to `v8::Value::ToString(v8::Isolate*) const'
collect2: error: ld returned 1 exit status
src/build.mk:335: recipe for target 'build/release/rethinkdb' failed
make[1]: *** [build/release/rethinkdb] Error 1
Makefile:52: recipe for target 'make' failed
make: *** [make] Error 2
```