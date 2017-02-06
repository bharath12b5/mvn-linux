<!---PACKAGE:rethinkDB--->
<!---DISTRO:Ubuntu 16.04--->

# Building RethinkDB v2.3.5

[RethinkDB](https://www.rethinkdb.com/) is the first open-source scalable database built for realtime applications. The instructions provided below specify the steps to build RethinkDB v2.3.5 on IBM z Systems for Ubuntu 16.04:

### _**General Note:**_
* _When following the steps below please use a standard permission user unless otherwise specified_
* _A directory \<source root\> will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it_

## Building and Installing RethinkDB v2.3.5

### Step 1: Install the dependencies
```
apt-get update
apt-get install build-essential protobuf-compiler python libprotobuf-dev libcurl4-openssl-dev libboost-all-dev libncurses5-dev wget m4 autoconf openjdk-8-jdk
```

Build `jemalloc 4.4.0` from source
```
git clone https://github.com/jemalloc/jemalloc.git
cd jemalloc
git checkout 4.4.0
./autogen.sh
make
make install_bin install_include install_lib
```

Install Node.js from https://developer.ibm.com/node/sdk/v12/. You need to download `Linux on System z 64-bit (file name: ibm-1.2.0.17-node-v0.12.18-linux-s390x.bin)`, then
```
chmod +x ibm-1.2.0.17-node-v0.12.18-linux-s390x.bin
./ibm-1.2.0.17-node-v0.12.18-linux-s390x.bin
export PATH=$PATH:/root/ibm/node/bin
npm install -g coffee-script@1.12.3
npm install -g browserify@12.0.1
```

### Step 2: Build and install RethinkDB
```
git clone https://github.com/rethinkdb/rethinkdb.git
cd rethinkdb
git checkout v2.3.5
```

Make changes to `./configure`
```diff
@@ -84,6 +84,8 @@ configure () {
         arm*)
             var_append LDFLAGS -ldl
             final_warning="ARM support is still experimental" ;;
+        s390x)
+            final_warning="s390x support is still experimental" ;;
         *)
             error "unsupported architecture: $MACHINE"
```

Then
```
./configure
```

Make changes to `./src/arch/runtime/context_switching.cc`
```diff
@@ -98,7 +98,7 @@ artificial_stack_t::artificial_stack_t(void (*initial_fun)(void), size_t _stack_
     the stack by swapcontext; they're callee-saved, so whatever happens to be in
     them will be ignored. */
     sp -= 6;
-#elif defined(__arm__)
+#elif defined(__arm__) || defined(__s390x__)
     /* We must preserve r4, r5, r6, r7, r8, r9, r10, and r11. Because we have to store the LR (r14) in swapcontext as well, we also store r12 in swapcontext to keep the stack double-word-aligned. However, we already accounted for both of those by decrementing sp twice above (once for r14 and once for r12, say). */
     sp -= 8;
#else
@@ -262,7 +262,7 @@ void context_switch(artificial_stack_context_ref_t *current_context_out, artific
}

asm(
-#if defined(__i386__) || defined(__x86_64__) || defined(__arm__)
+#if defined(__i386__) || defined(__x86_64__) || defined(__arm__) || defined(__s390x__)
// We keep the i386, x86_64, and ARM stuff interleaved in order to enforce commonality.
#if defined(__x86_64__)
#if defined(__LP64__) || defined(__LLP64__)
@@ -281,6 +281,7 @@ asm(
     /* `current_pointer_out` is in `%rdi`. `dest_pointer` is in `%rsi`. */
#elif defined(__arm__)
     /* `current_pointer_out` is in `r0`. `dest_pointer` is in `r1` */
+#elif defined(__s390x__)
#endif

     /* Save preserved registers (the return address is already on the stack). */
@@ -302,6 +303,8 @@ asm(
     "push {r12}\n"
     "push {r14}\n"
     "push {r4-r11}\n"
+#elif defined(__s390x__)
+    ""
#endif

     /* Save old stack pointer. */
@@ -316,6 +319,8 @@ asm(
#elif defined(__arm__)
     /* On ARM, the first argument is in `r0`. `r13` is the stack pointer. */
     "str r13, [r0]\n"
+#elif defined(__s390x__)
+    ""
#endif

     /* Load the new stack pointer and the preserved registers. */
@@ -330,6 +335,8 @@ asm(
#elif defined(__arm__)
     /* On ARM, the second argument is in `r1` */
     "mov r13, r1\n"
+#elif defined(__s390x__)
+    ""
#endif

#if defined(__i386__)
@@ -348,6 +355,8 @@ asm(
     "pop {r4-r11}\n"
     "pop {r14}\n"
     "pop {r12}\n"
+#elif defined(__s390x__)
+    ""
#endif

#if defined(__i386__) || defined(__x86_64__)
@@ -360,6 +369,8 @@ asm(
     /* Above, we popped `LR` (`r14`) off the stack, so the bx instruction will
     jump to the correct return address. */
     "bx r14\n"
+#elif defined(__s390x__)
+    ""
#endif

#else
```

Make changes to `./src/rdb_protocal/datum.cc`
```diff
@@ -1118,9 +1118,10 @@ std::string datum_t::mangle_secondary(
std::string datum_t::encode_tag_num(uint64_t tag_num) {
     static_assert(sizeof(tag_num) == tag_size,
             "tag_size constant is assumed to be the size of a uint64_t.");
-#ifndef BOOST_LITTLE_ENDIAN
+/*#ifndef BOOST_LITTLE_ENDIAN
     static_assert(false, "This piece of code will break on big-endian systems.");
#endif
+*/
     return std::string(reinterpret_cast<const char *>(&tag_num), tag_size);
}

@@ -1244,9 +1245,10 @@ components_t parse_secondary(const std::string &key) THROWS_NOTHING {
     std::string tag_str = key.substr(start_of_tag, key.size() - (start_of_tag + 2));
     boost::optional<uint64_t> tag_num;
     if (tag_str.size() != 0) {
-#ifndef BOOST_LITTLE_ENDIAN
+/*#ifndef BOOST_LITTLE_ENDIAN
         static_assert(false, "This piece of code will break on little endian systems.");
#endif
+*/
         tag_num = *reinterpret_cast<const uint64_t *>(tag_str.data());
     }
     return components_t{
```

Make changes to `./src/rpc/connectivity/cluster.cc`
```diff
@@ -103,7 +103,7 @@ static bool resolve_protocol_version(const std::string &remote_version_string,
     return false;
}

-#if defined (__x86_64__) || defined (_WIN64)
+#if defined (__x86_64__) || defined (_WIN64) || defined (__s390x__)
const std::string connectivity_cluster_t::cluster_arch_bitsize("64bit");
#elif defined (__i386__) || defined(__arm__)
const std::string connectivity_cluster_t::cluster_arch_bitsize("32bit");
```

Install v8-3.28 for s390x, following https://github.com/linux-on-ibm-z/docs/wiki/Building-V8-libraries-3.x. Assuming you have finished the building and installation, and the install directory is `/<source_root>/v8-3.28-z`
```
mkdir /<source_root>/rethinkdb/external/v8_3.30.33.16/
cp -RL /<source_root>/v8-3.28-z/* /<source_root>/rethinkdb/external/v8_3.30.33.16/
```

Make changes to `/usr/include/libplatform/libpliatform.h`,
```diff
-#include "include/v8-platform.h"
+#include "v8-platform.h"
```

Make changes to `mk/support/pkg/v8.sh`
```diff
@@ -42,6 +42,7 @@ pkg_install () {
         i?86)   arch=ia32 ;;
         x86_64) arch=x64 ;;
         arm*)   arch=arm; arch_gypflags=$raspberry_pi_gypflags ;;
+        s390x)  arch=s390x ;;
         *)      arch=native ;;
     esac
     mode=release

```

Make changes to `src/build.mk`
```diff
@@ -53,7 +53,7 @@ else ifeq ($(COMPILER),INTEL)
else ifeq ($(COMPILER),GCC)

   ifeq ($(OS),Linux)
-    RT_LDFLAGS += -Wl,--no-as-needed
+    RT_LDFLAGS += -lv8_base -lv8_libbase -lv8_libplatform -lv8_nosnapshot -lv8_snapshot -licui18n -licuuc -ldl -Wl,--no-as-needed
   endif

   ifeq ($(STATICFORCE),1)
```

Make changes to `src/client_protocol/server.cc`
```diff
@@ -280,6 +280,7 @@ void query_server_t::handle_conn(const scoped_ptr_t<tcp_conn_descriptor_t> &ncon
         int32_t client_magic_number;
         conn->read_buffered(
             &client_magic_number, sizeof(client_magic_number), &ct_keepalive);
+        client_magic_number = __builtin_bswap32(client_magic_number);

         switch (client_magic_number) {
             case VersionDummy::V0_1:
```

Make changes to `src/perfmon/perfmon.cc`
```diff
@@ -159,7 +159,7 @@ stddev_t::stddev_t()
stddev_t::stddev_t(size_t n, double mean, double variance)
     : N(n), M(mean), Q(variance * n) {
     if (N == 0)
-        rassert(isnan(M) && isnan(Q));
+        rassert(std::isnan(M) && std::isnan(Q));
}
```

Then, `n` is the total number of your CPU cores
```
make -j <n> THREADED_COROUTINES=1
make install -j <n> THREADED_COROUTINES=1
```

To start a server
```
rethinkdb --bind all
```

For unit testing
```
make -j <n> THREADED_COROUTINES=1 DEBUG=1
./test/run unit -j <n>
```
 