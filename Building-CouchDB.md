<!---PACKAGE:CouchDB--->
<!---DISTRO:RHEL 6.x:1.6.1--->
<!---DISTRO:RHEL 7.x:1.6.1--->
<!---DISTRO:SLES 11.x:1.6.1--->
<!---DISTRO:SLES 12.x:1.6.1--->
<!---DISTRO:Ubuntu 16.x:Distro,1.6.1--->


#Building CouchDB

Below versions of CouchDB are available in respective distributions at the time of this recipe creation:

* Ubuntu 16.04 has  1.6.0


The instructions provided below specify the steps to build CouchDB version 2.0.0 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.




### _**General Note:**_
 * _When following the steps below please use a standard permission user unless otherwise specified._  
 * _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_  


### Prerequisites

* Erlang 19.2 (RHEL & SLES)
  -- Instructions for building Erlang 19.2 can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang)
  
##Step 1 : Building CouchDB
####1.1) Install dependencies  
•	RHEL 6.8   
```
sudo yum install -y libicu-devel libcurl libcurl-devel git wget zip bzip2 tar nspr nspr-devel autoconf213 fontconfig fontconfig-devel pkgconfig make gtk2-devel libXt-devel libIDL libIDL-devel freetype freetype-devel gstreamer1 gstreamer1-devel dbus dbus-devel dbus-glib dbus-glib-devel libnotify libnotify-devel python perl xz libtool which gcc-c++ curl
```  

•	RHEL 7.1/7.2/7.3  
```
sudo yum install -y libicu-devel libcurl libcurl-devel git wget zip bzip2 tar nspr nspr-devel autoconf213 fontconfig fontconfig-devel pkgconfig make gtk2-devel libXt-devel libIDL libIDL-devel freetype freetype-devel gstreamer1 gstreamer1-devel dbus dbus-devel dbus-glib dbus-glib-devel libnotify libnotify-devel python perl xz libtool which gcc-c++ curl
```  

•	SLES 11-SP4  
```
sudo zypper install -y libicu-devel libcurl4 libcurl-devel git wget autoconf213 zip fontconfig-devel pkg-config  yasm tar make gtk2-devel  dbus-1-devel dbus-1-glib-devel  libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel libtool pkg-config curl automake libidl-devel libnotify-devel fontconfig libasound2 python wget perl perl-base
```   

•	SLES 12/12-SP1/12-SP2  
```
sudo zypper install libicu-devel libcurl4 libcurl-devel git wget mozilla-nspr mozilla-nspr-devel autoconf213 zip fontconfig fontconfig-devel pkg-config libasound2 yasm tar make gtk2-devel libXt-devel libIDL-2-0 libidl-devel libfreetype6 libgstreamer-1_0-0 dbus-1-devel dbus-1-glib-devel libnotify4 libnotify-devel python wget perl perl-base libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel libtool pkg-config gcc-c++ curl
```  

•	Ubuntu 16.04  
```
sudo apt-get update
sudo apt-get install -y libicu55 erlang erlang-dev rebar ncurses-base gcc python git wget tar make autoconf2.13 automake autoconf g++ libmozjs185-1.0 libmozjs185-dev  openssl    libicu-dev libcurl4-openssl-dev  build-essential unixodbc libncurses5-dev libncursesw5-dev fop libncurses5-dev unixodbc-dev libssl-dev pkg-config curl
```

•	Ubuntu 16.10  
```
sudo apt-get update
sudo apt-get install -y libicu57 erlang erlang-dev rebar ncurses-base gcc python git wget tar make autoconf2.13 automake autoconf g++ libmozjs185-1.0 libmozjs185-dev  openssl    libicu-dev libcurl4-openssl-dev  build-essential unixodbc libncurses5-dev libncursesw5-dev fop libncurses5-dev unixodbc-dev libssl-dev pkg-config curl
```
      



####1.2) Install gcc-c++ for SLES 11-SP4	    
```
sudo zypper install -y gcc47-c++
sudo ln -s /usr/bin/gcc-4.7 /usr/bin/gcc  
sudo ln -s /usr/bin/gcc-4.7 /usr/bin/cc
sudo ln -s /usr/bin/g++-4.7 /usr/bin/g++
sudo ln -s /usr/bin/g++-4.7 /usr/bin/c++
```


####1.3) Install SpiderMonkey (Only for RHEL/SLES)

•	Obtain the [SpiderMonkey](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Releases/1.8.5) and extract the source   
```
cd /<source_root>/
wget http://ftp.mozilla.org/pub/mozilla.org/js/js185-1.0.0.tar.gz
tar zxf js185-1.0.0.tar.gz
```

•	Edit file `/<source_root>/js-1.8.5/js/src/jsval.h`
        
```diff

@@ -307,11 +307,11 @@
            uint32         u32;
            JSWhyMagic     why;
-           jsuword       word;
        } payload;
      } s;
             double asDouble;
             void *asPtr;
+            jsuword asWord;
        } jsval_layout;
@@ -321,6 +321,7 @@
      struct {
              JSValueTag tag;
+             uint32 padding;
       union {
@@ -344,6 +345,7 @@
       } debugView;
       struct {
+             uint32 padding;
              union {
                       int32 i32;
                       uint32 u32;
@@ -352,6 +354,7 @@
              double asDouble;
              void *asPtr;
+             jsuword asWord;
      } jsval_layout;
```

•	Edit file `/<source_root>/js-1.8.5/js/src/jsvalue.h`
        
```diff

@@ -292,7 +292,6 @@

      #ifdef __cplusplus
-     JS_STATIC_ASSERT(offsetof(jsval_layout, s.payload) == 0);
      JS_STATIC_ASSERT((JSVAL_TYPE_NONFUNOBJ & 0xF) == JSVAL_TYPE_OBJECT);
@@ -730,7 +729,11 @@ class Value

      const jsuword *payloadWord() const {
-        return &data.s.payload.word;
+    #if JS_BITS_PER_WORD == 32 
+   	  return reinterpret_cast<const jsuword *>(&data.s.payload.word);
+	 #elif JS_BITS_PER_WORD == 64 
+	  return reinterpret_cast<const jsuword *>(&data.asBits);
+	 #endif
                  }
```

•	Edit file `/<source_root>/js-1.8.5/js/src/Makefile.in`
        
```diff

@@ -383,7 +381,7 @@
#############################################

-ifeq (,$(filter-out powerpc sparc,$(TARGET_CPU)))
+ifeq (,$(filter arm %86 x86_64,$(TARGET_CPU)))

VPATH +=	$(srcdir)/assembler \
```


•	Prepare the source code    
```
cd /<source_root>/js-1.8.5/js/src
autoconf-2.13
```
•	Configure, build & install SpiderMonkey    
```
mkdir /<source_root>/js-1.8.5/js/src/build_OPT.OBJ
cd /<source_root>/js-1.8.5/js/src/build_OPT.OBJ
../configure --prefix=/usr
make
sudo make install
```

####1.4) Build and install Rebar (Only for RHEL/SLES)

•	Build rebar
```
cd /<source_root>/
git clone git://github.com/rebar/rebar.git
cd /<source_root>/rebar
./bootstrap
```
     
•	The bootstrap command produces an Erlang escript named rebar. Copy it into a directory in your `$PATH`:
```
sudo cp /<source_root>/rebar/rebar /usr/local/bin/
```
     
####1.5) Download the CouchDB source code
```
cd /<source_root>/
git clone https://git-wip-us.apache.org/repos/asf/couchdb.git
cd /<source_root>/couchdb
git checkout 2.0.0
```
     
####1.6) Configure and build CouchDB
```
./configure -c --disable-docs --disable-fauxton
export LD_LIBRARY_PATH=/usr/lib
make		
```
 
## Step 2: Testing (Optional)
####2.1) Run all test cases  
```
make check
```


    
_**Notes:**_
	
•	_Three test case failures from module test/couch_compress_tests.erl at Line 35,49 & 51 are observed, due to snappy behavior on z System. Modify the file as shown below to pass the test cases:_
      
Edit file `/<source_root>/couchdb/src/couch/test/couch_compress_tests.erl`

```diff
@@ -22,8 +22,8 @@
--define(SNAPPY, <<1,49,64,131,104,1,108,0,0,0,5,104,2,100,0,
-    1,97,97,1,104,1,8,8,98,97,2,5,8,8,99,97,3,5,8,44,100,97,
+-define(SNAPPY, <<1,49,60,131,104,1,108,0,0,0,5,104,2,100,0,
+    1,97,97,1,5,8,8,98,97,2,5,8,8,99,97,3,5,8,44,100,97,
     4,104,2,100,0,1,101,97,5,106>>).
```
    
•	 _For RHEL 6.8 and SLES distributions, if test case fails with error `libmozjs185.so.1.0 not found`, copy libmozjs185.so.1.0 to /lib64 as follows:_

```
sudo cp /usr/lib/libmozjs185.so.1.0 /usr/lib64/libmozjs185.so.1.0
```
•	_For RHEL 7 distributions, if test case fails with error `libmozjs185.so.1.0 not found`, copy libmozjs185.so.1.0 to /lib as follows:_

```
sudo cp /usr/lib64/libmozjs185.so.1.0 /usr/lib/libmozjs185.so.1.0
```

####2.2) Start the CouchDB server

```
sudo /<source_root>/couchdb/dev/run &
```
 
####2.3) Verify the output of CouchDB server by using `curl http://127.0.0.1:15984/` command. The output should be as follows:  
```
{"couchdb":"Welcome","uuid"="xxx","version":"2.0","vendor":{"name":"The Apache Software Foundation"}}
```

### References:
[CouchDB](http://couchdb.apache.org/)

[CouchDB 2.0 Documentation](http://docs.couchdb.org/en/2.0.0/)