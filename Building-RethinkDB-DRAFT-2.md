<!---PACKAGE:rethinkDB--->
<!---DISTRO:SLES 12 SP2--->

# Building RethinkDB v2.3.5

[rethinkDB](https://www.rethinkdb.com/) RethinkDB is the first open-source scalable database built for realtime applications. The instructions provided below specify the steps to build RethinkDB v2.3.5 on IBM z Systems for Ubuntu 16.04:

### _**General Note:**_
* _When following the steps below please use a standard permission user unless otherwise specified_
* _A directory \<source root\> will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it_

## Building and Installing RethinkDB v2.3.5

### Step 1: Install the dependencies
```
apt-get install build-essential protobuf-compiler python libprotobuf-dev libcurl4-openssl-dev libboost-all-dev libncurses5-dev libjemalloc-dev wget m4
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

Install Node.js from https://developer.ibm.com/node/sdk/. You need to download `ibm-1.2.0.17-node-v0.12.18-linux-s390x.bin`, then
```
chmod +x ibm-6.9.4.0-node-v6.9.4-linux-s390x.bin
./ibm-6.9.4.0-node-v6.9.4-linux-s390x.bin
export PATH=$PATH:/root/ibm/node/bin
npm install -g coffee-script@1.12.3
npm install -g browserify@12.0.1
```

Build `protobuf 2.5.0` by following recipe in https://github.com/linux-on-ibm-z/docs/wiki/Building-Google-Protobuf-2.5.0


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
@@ -101,6 +101,8 @@ artificial_stack_t::artificial_stack_t(void (*initial_fun)(void), size_t _stack_
#elif defined(__arm__)
     /* We must preserve r4, r5, r6, r7, r8, r9, r10, and r11. Because we have to store the LR (r14) in swapcontext as well, we also store r12 in swapcontext to keep the stack double-word-aligned. However, we already accounted for both of those by decrementing sp twice above (once for r14 and once for r12, say). */
     sp -= 8;
+#elif defined(__s390x__)
+    sp -= 10;
#else
#error "Unsupported architecture."
#endif
@@ -262,7 +264,7 @@ void context_switch(artificial_stack_context_ref_t *current_context_out, artific
}

asm(
-#if defined(__i386__) || defined(__x86_64__) || defined(__arm__)
+#if defined(__i386__) || defined(__x86_64__) || defined(__arm__) || defined(__s390x__)
// We keep the i386, x86_64, and ARM stuff interleaved in order to enforce commonality.
#if defined(__x86_64__)
#if defined(__LP64__) || defined(__LLP64__)
@@ -281,6 +283,7 @@ asm(
     /* `current_pointer_out` is in `%rdi`. `dest_pointer` is in `%rsi`. */
#elif defined(__arm__)
     /* `current_pointer_out` is in `r0`. `dest_pointer` is in `r1` */
+#elif defined(__s390x__)
#endif

     /* Save preserved registers (the return address is already on the stack). */
@@ -302,6 +305,13 @@ asm(
     "push {r12}\n"
     "push {r14}\n"
     "push {r4-r11}\n"
+#elif defined(__s390x__)
+    ""
+/*    "stmg %r6,%13,-88(%r15)\n"
+    "stg %r15,-24(%r15)\n"
+    "stg %r14,-16(%r15)"
+    "aghi %r15,-88\n"
+*/
#endif

     /* Save old stack pointer. */
@@ -316,6 +326,10 @@ asm(
#elif defined(__arm__)
     /* On ARM, the first argument is in `r0`. `r13` is the stack pointer. */
     "str r13, [r0]\n"
+#elif defined(__s390x__)
+    ""
+/*    "stg %r15,0(%r2)\n"
+*/
#endif

@@ -330,6 +344,10 @@ asm(
#elif defined(__arm__)
     /* On ARM, the second argument is in `r1` */
     "mov r13, r1\n"
+#elif defined(__s390x__)
+    ""
+/*    "lgr %r15,%r3\n"
+*/
#endif

#if defined(__i386__)
@@ -348,6 +366,12 @@ asm(
     "pop {r4-r11}\n"
     "pop {r14}\n"
     "pop {r12}\n"
+#elif defined(__s390x__)
+    ""
+/*    "lmg %r14,72(%r15)\n"
+    "lmg %r6,%r13,0(%r15)\n"
+    "lmg %r15,64(%r15)\n"
+*/
#endif

#if defined(__i386__) || defined(__x86_64__)
@@ -360,6 +384,8 @@ asm(
     /* Above, we popped `LR` (`r14`) off the stack, so the bx instruction will
     jump to the correct return address. */
     "bx r14\n"
+#elif defined(__s390x__)
+    ""
#endif
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

Install v8-3.28 for s390x, follow https://github.com/linux-on-ibm-z/docs/wiki/Building-V8-libraries-3.x. Assuming you have finished the building and installation, and the install directory is `/<source_root>/v8-3.28-z`
```
cp -RL /<source_root>/v8-3.28-z/* /<source_root>/rethinkdb/external/v8_3.30.33.16/
```

Make changes to `/usr/include/libplatform/libpliatform.h`,
```diff
-#include "include/v8-platform.h"
+#include "v8-platform.h"
```

Rewrite `mk/support/pkg/v8.sh` as following
```
version=3.30.33.16

pkg_install-include () {
}

pkg_install () {
}

pkg_link-flags () {
}
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

Then
```
make
make install
```

For unit testing
```
./test/run unit
```
 