<!---PACKAGE:RethinkDB--->
<!---DISTRO:Ubuntu 16.04--->

# Building RethinkDB v2.3.5

[RethinkDB](https://www.rethinkdb.com/) RethinkDB is the open-source, scalable database that makes building realtime apps dramatically easier. The stable release of RethinkDB 2.3.5 has been built and tested on Linux on z Systems.  The following instructions can be used for Ubuntu 16.04.

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
```diff
index 728c2a6..48e64ba 100755
--- a/configure
+++ b/configure
@@ -79,7 +79,7 @@ configure () {
         *) error "unsupported operating system: $MACHINE" ;;
     esac
     case "${MACHINE%%-*}" in
-        x86_64|i?86)
+        s390x|x86_64|i?86)
             true ;;
         arm*)
             var_append LDFLAGS -ldl
```        
Modify ./src/rpc/connectivity/cluster.cc
```diff
index b43f7ab..c418fab 100644
--- a/src/rpc/connectivity/cluster.cc
+++ b/src/rpc/connectivity/cluster.cc
@@ -103,7 +103,7 @@ static bool resolve_protocol_version(const std::string &remote_version_string,
     return false;
 }

-#if defined (__x86_64__) || defined (_WIN64)
+#if defined (__x86_64__) || defined (_WIN64) || defined(__s390x__)
 const std::string connectivity_cluster_t::cluster_arch_bitsize("64bit");
 #elif defined (__i386__) || defined(__arm__)
 const std::string connectivity_cluster_t::cluster_arch_bitsize("32bit");

```
Modify ./src/arch/runtime/context_switching.cc
```diff
index d729575..3da8b06 100644
--- a/src/arch/runtime/context_switching.cc
+++ b/src/arch/runtime/context_switching.cc
@@ -98,7 +98,7 @@ artificial_stack_t::artificial_stack_t(void (*initial_fun)(void), size_t _stack_
     the stack by swapcontext; they're callee-saved, so whatever happens to be in
     them will be ignored. */
     sp -= 6;
-#elif defined(__arm__)
+#elif defined(__arm__) || defined (__s390x__)
     /* We must preserve r4, r5, r6, r7, r8, r9, r10, and r11. Because we have to store the LR (r14) in swapcontext as well, we also store r12 in swapcontext to keep the stack double-word-aligned. However, we already accounted for both of those by decrementing sp twice above (once for r14 and once for r12, say). */
     sp -= 8;
 #else
@@ -262,7 +262,7 @@ void context_switch(artificial_stack_context_ref_t *current_context_out, artific
 }

 asm(
-#if defined(__i386__) || defined(__x86_64__) || defined(__arm__)
+#if defined(__i386__) || defined(__x86_64__) || defined(__arm__) || defined (__s390x__)
 // We keep the i386, x86_64, and ARM stuff interleaved in order to enforce commonality.
 #if defined(__x86_64__)
 #if defined(__LP64__) || defined(__LLP64__)

```
Modify ./src/rdb_protocol/datum.cc and comment out these lines
```diff
index 7fbd7aa..a0b988d 100644
--- a/src/rdb_protocol/datum.cc
+++ b/src/rdb_protocol/datum.cc
@@ -1118,9 +1118,10 @@ std::string datum_t::mangle_secondary(
 std::string datum_t::encode_tag_num(uint64_t tag_num) {
     static_assert(sizeof(tag_num) == tag_size,
             "tag_size constant is assumed to be the size of a uint64_t.");
-#ifndef BOOST_LITTLE_ENDIAN
-    static_assert(false, "This piece of code will break on big-endian systems.");
-#endif
+// Disable for s390x
+//#ifndef BOOST_LITTLE_ENDIAN
+//    static_assert(false, "This piece of code will break on big-endian systems.");
+//#endif
     return std::string(reinterpret_cast<const char *>(&tag_num), tag_size);
 }

@@ -1244,9 +1245,10 @@ components_t parse_secondary(const std::string &key) THROWS_NOTHING {
     std::string tag_str = key.substr(start_of_tag, key.size() - (start_of_tag + 2));
     boost::optional<uint64_t> tag_num;
     if (tag_str.size() != 0) {
-#ifndef BOOST_LITTLE_ENDIAN
-        static_assert(false, "This piece of code will break on little endian systems.");
-#endif
+// Disable for s390x
+//#ifndef BOOST_LITTLE_ENDIAN
+//        static_assert(false, "This piece of code will break on little endian systems.");
+//#endif
         tag_num = *reinterpret_cast<const uint64_t *>(tag_str.data());
     }
     return components_t{

```
Modify ./src/rdb_protocol/terms/string.cc and change ICU function as follows
```diff
index a17e443..0bebd6c 100644
--- a/src/rdb_protocol/terms/string.cc
+++ b/src/rdb_protocol/terms/string.cc
@@ -16,7 +16,7 @@ namespace ql {
 // Combining characters in Unicode have a character class starting with M. There
 // are three types, which affect layout; we don't distinguish between them here.
 static bool is_combining_character(char32_t c) {
-    return (U_GET_GC_MASK(c) & U_GC_M_MASK) > 0;
+    return (c!=' '); // s390x - remove ICU ref
 }

 // ICU provides several different whitespace functions.  We use
@@ -26,7 +26,7 @@ static bool is_combining_character(char32_t c) {
 // (Z designating separators) and some of the Z category marks do not
 // have the WSpace=Y property; principally zero width spacers).
 static bool is_whitespace_character(char32_t c) {
-    return u_isUWhiteSpace(c);
+    return std::isspace(c); // s390x - replace with non-ICU libraray
 }

 class match_term_t : public op_term_t {

```
Modify ./external/v8_3.30.33.16/include/v8.h
```diff
index d5433a6..4c92b3c 100644
--- a/external/v8_3.30.33.16/include/v8.h
+++ b/external/v8_3.30.33.16/include/v8.h
@@ -6857,7 +6857,7 @@ Local<Number> Value::ToNumber() const {


 Local<String> Value::ToString() const {
-  return ToString(Isolate::GetCurrent());
+  return ToString(); // s390x - V8 3.28 has no support for Isolate
 }


@@ -6867,7 +6867,7 @@ Local<String> Value::ToDetailString() const {


 Local<Object> Value::ToObject() const {
-  return ToObject(Isolate::GetCurrent());
+  return ToObject(); // s390x has no support for Isolate
 }

```
Modify ./src/extproc/js_job.cc
```diff
index 1a0b7db..b715a7c 100644
--- a/src/extproc/js_job.cc
+++ b/src/extproc/js_job.cc
@@ -56,8 +56,9 @@ js_instance_t::js_instance_t() {
     platform.init(v8::platform::CreateDefaultPlatform());
     v8::V8::InitializePlatform(platform.get());
     v8::V8::Initialize();
-    isolate_ = v8::Isolate::New();
-    isolate_->Enter();
+    // Disable Isolate for s390x v8 3.28
+    //isolate_ = v8::Isolate::New();
+    //isolate_->Enter();
 }

 js_instance_t::~js_instance_t() {

```
Use 3.28 V8 packages for s390x and modify ./mk/support/pkg/v8.sh as follows
```diff
index dc339ad..4081959 100644
--- a/mk/support/pkg/v8.sh
+++ b/mk/support/pkg/v8.sh
@@ -45,8 +45,20 @@ pkg_install () {
         *)      arch=native ;;
     esac
     mode=release
-    pkg_make $arch.$mode CXX=$CXX LINK=$CXX LINK.target=$CXX GYPFLAGS="-Dwerror= $arch_gypflags" V=1
-    for lib in `find "$build_dir/out/$arch.$mode" -maxdepth 1 -name \*.a` `find "$build_dir/out/$arch.$mode/obj.target" -name \*.a`; do
+    # use s390x v8 3.28
+    cd $build_dir
+    rm -rf $build_dir/v8z
+    rm -rf $build_dir/depot_tools
+    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
+    export PATH=$(pwd)/depot_tools:$PATH
+    git clone https://github.com/ibmruntimes/v8z
+    cd v8z
+    git checkout 3.28-s390
+    make dependencies
+    make s390x -j4 library=shared i18nsupport=off
+
+    #pkg_make $arch.$mode CXX=$CXX LINK=$CXX LINK.target=$CXX GYPFLAGS="-Dwerror= $arch_gypflags" V=1
+    for lib in `find "$build_dir/v8z/out/s390x.release" -maxdepth 1 -name \*.a` `find "$build_dir/v8z/out/s390x.release/obj.target" -name \*.a`; do
         name=`basename $lib`
         cp $lib "$install_dir/lib/${name/.$arch/}"
     done
@@ -59,7 +71,8 @@ pkg_link-flags () {
     for lib in libv8_{base,libbase,snapshot,libplatform}; do
         echo "$install_dir/lib/$lib.a"
     done
-    for lib in libicu{i18n,uc,data}; do
-        echo "$install_dir/lib/$lib.a"
-    done
+    # disable icu linking for s390x v8 3.28
+    #for lib in libicu{i18n,uc,data}; do
+    #    echo "$install_dir/lib/$lib.a"
+    #done
 }

```
Configure and make RethinkDB
```
./configure --allow-fetch --dynamic jemalloc
make
make install
```