<!---PACKAGE:CouchDB--->
<!---DISTRO:RHEL 6.6:1.6.1--->
<!---DISTRO:RHEL 7.1:1.6.1--->
<!---DISTRO:SLES 11:1.6.1--->
<!---DISTRO:SLES 12:1.6.1--->
<!---DISTRO:Ubuntu 16.x:Distro,1.6.1--->

<!---PACKAGE:CouchDB--->
<!---DISTRO:RHEL 6.6:1.6.1--->
<!---DISTRO:RHEL 7.1:1.6.1--->
<!---DISTRO:SLES 11:1.6.1--->
<!---DISTRO:SLES 12:1.6.1--->
<!---DISTRO:Ubuntu 16.x:Distro--->

#Building CouchDB

Below versions of CouchDB are available in respective distributions at the time of this recipe creation:

* Ubuntu 16.04 has  1.6.0


The instructions provided below specify the steps to build CouchDB Version 2.0.0 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.




### _**General Note:**_
 * _When following the steps below please use a standard permission user unless otherwise specified._  
 * _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_  


### Prerequisites

* Erlang 19.0 (RHEL & SLES)
  -- Instructions for building Erlang 19.0 can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang)
  
### Building and Installing CouchDB
1. Install the build dependencies

    RHEL 6.8
    
        sudo yum install -y libicu-devel libcurl libcurl-devel git wget zip bzip2 tar nspr nspr-devel autoconf213 fontconfig fontconfig-devel pkgconfig make gtk2-devel libXt-devel libIDL libIDL-devel freetype freetype-devel gstreamer1 gstreamer1-devel dbus dbus-devel dbus-glib dbus-glib-devel libnotify libnotify-devel python perl xz libtool which gcc-c++ curl


	RHEL 7.1/7.2/7.3


        sudo yum install -y libicu-devel libcurl libcurl-devel git wget zip bzip2 tar nspr nspr-devel autoconf213 fontconfig fontconfig-devel pkgconfig make gtk2-devel libXt-devel libIDL libIDL-devel freetype freetype-devel gstreamer1 gstreamer1-devel dbus dbus-devel dbus-glib dbus-glib-devel libnotify libnotify-devel python perl xz libtool which gcc-c++ curl




	SLES 11-SP4
    
        sudo zypper install -y libicu-devel libcurl4 libcurl-devel git wget autoconf213 zip fontconfig-devel pkg-config  yasm tar make gtk2-devel  dbus-1-devel dbus-1-glib-devel  libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel libtool pkg-config curl automake libidl-devel libnotify-devel fontconfig libasound2 python wget perl perl-base

	SLES 12/12-SP1/12-SP2

        sudo zypper install libicu-devel libcurl4 libcurl-devel git wget mozilla-nspr mozilla-nspr-devel autoconf213 zip fontconfig fontconfig-devel pkg-config libasound2 yasm tar make gtk2-devel libXt-devel libIDL-2-0 libidl-devel libfreetype6 libgstreamer-1_0-0 dbus-1-devel dbus-1-glib-devel libnotify4 libnotify-devel python wget perl perl-base libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel libtool pkg-config gcc-c++ curl
       

	Ubuntu 16.04

        sudo apt-get update
        sudo apt-get install -y libicu55 erlang erlang-dev rebar phantomjs ncurses-base gcc python git wget tar make autoconf2.13 automake autoconf g++ libmozjs185-1.0 libmozjs185-dev  openssl    libicu-dev libcurl4-openssl-dev  build-essential unixodbc libncurses5-dev libncursesw5-dev fop libncurses5-dev unixodbc-dev libssl-dev pkg-config curl

	Ubuntu 16.10

        sudo apt-get update
        sudo apt-get install -y libicu57 erlang erlang-dev rebar phantomjs ncurses-base gcc python git wget tar make autoconf2.13 automake autoconf g++ libmozjs185-1.0 libmozjs185-dev  openssl    libicu-dev libcurl4-openssl-dev  build-essential unixodbc libncurses5-dev libncursesw5-dev fop libncurses5-dev unixodbc-dev libssl-dev pkg-config curl



2. Install gcc-c++

	SLES 11-SP4

        sudo zypper install -y gcc47-c++
        sudo ln -s /usr/bin/gcc-4.7 /usr/bin/gcc
        sudo ln -s /usr/bin/gcc-4.7 /usr/bin/cc
        sudo ln -s /usr/bin/g++-4.7 /usr/bin/g++
        sudo ln -s /usr/bin/g++-4.7 /usr/bin/c++



2. Install SpiderMonkey (Only for RHEL/SLES)

    * Obtain the [SpiderMonkey](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Releases/1.8.5) source code
        ```
        cd /<source_root>/
        wget http://ftp.mozilla.org/pub/mozilla.org/js/js185-1.0.0.tar.gz
        ```
    
    * Untar the archive and change working directory
        ```
        tar zxf js185-1.0.0.tar.gz && cd js-1.8.5/js/src
        ```

    * Effect the following source code changes

        _**NOTE:** The given line numbers account for any previous action on the file._

            Edit file /<source_root>/js-1.8.5/js/src/jsval.h
            Remove line 309 ( content: 'jsuword        word;' )
            Add a new line after line number 312 and insert the following string:
                jsuword        asWord;
            Add a new line after line number 322 and insert the following string:
                uint32         padding;
            Add a new line after line number 346 and insert the following string:
                uint32         padding;
            Add a new line after line number 355 and insert the following string:
                jsuword        asWord;
    Save the file.

        _For more info on the above see the [bugzilla diff](
    http://hg.mozilla.org/mozilla-central/rev/6f2c0dbb88d3)_

            Edit file /<source_root>/js-1.8.5/js/src/jsvalue.h
            Remove line 294 ( content: 'JS_STATIC_ASSERT(offsetof(jsval_layout, s.payload) == 0);' )
            Replace line 731 ( content: 'return &data.s.payload.word;' )
            with the following content:
                #if JS_BITS_PER_WORD == 32
                    return reinterpret_cast<const jsuword *>(&data.s.payload.word);
                #elif JS_BITS_PER_WORD == 64
                    return reinterpret_cast<const jsuword *>(&data.asBits);
                #endif
        Save the file.

        _For more info on the above see the [bugzilla diff](
    https://bugzilla.mozilla.org/attachment.cgi?id=517107&action=diff)_

            Edit file /<source_root>/js-1.8.5/js/src/Makefile.in
            Change line number 385:
                From: ifeq (,$(filter-out powerpc sparc,$(TARGET_CPU)))
                To:   ifeq (,$(filter arm %86 x86_64,$(TARGET_CPU)))
        Save the file.

        _For more info on the above see the [bugzilla diff](https://bugzilla.mozilla.org/attachment.cgi?id=520157&action=diff)_

    * Prepare the src code using autoconf

            autoconf-2.13

    * Create a workspace directory and cd into it

            mkdir /<source_root>/js-1.8.5/js/src/build_OPT.OBJ
            cd /<source_root>/js-1.8.5/js/src/build_OPT.OBJ

    * Run the Configure step

            ../configure --prefix=/usr

    * Go ahead and make SpiderMonkey

            make

    * If no error is found, install SpiderMonkey

            sudo make install


3. Build and install Rebar (Only for RHEL/SLES)

     * CouchDB requires `rebar`, build it from source by running

            cd /<source_root>/
            git clone git://github.com/rebar/rebar.git
            cd /<source_root>/rebar
            ./bootstrap

    * The bootstrap command produces an Erlang escript named rebar. Copy it into a directory in your `$PATH`:

            sudo cp /<source_root>/rebar/rebar /usr/local/bin/
		
4. Now set up the CouchDB source tree

        cd /<source_root>/
        git clone https://git-wip-us.apache.org/repos/asf/couchdb.git
        cd /<source_root>/couchdb
        git checkout 2.0.0

5. Configure the source tree (which will fetch more source code from version control), then run make to build CouchDB
     ```
    ./configure -c --disable-docs --disable-fauxton
    export LD_LIBRARY_PATH=/usr/lib
    make		
    ```
 
6. Now make and run some sanity tests

     ````
    make check
     ````


    
   _**Note:**_
	
    * _Three test case failures from module test/couch_compress_tests.erl at Line 35,49 & 51 are observed, due to snappy behavior on z System. Modify the file as shown below to pass the test cases:_
      
     Edit file `/<source_root>/couchdb/src/couch/test/couch_compress_tests.erl`

     ```diff
     @@ -22,8 +22,8 @@
     --define(SNAPPY, <<1,49,64,131,104,1,108,0,0,0,5,104,2,100,0,
     -    1,97,97,1,104,1,8,8,98,97,2,5,8,8,99,97,3,5,8,44,100,97,
     +-define(SNAPPY, <<1,49,60,131,104,1,108,0,0,0,5,104,2,100,0,
     +    1,97,97,1,5,8,8,98,97,2,5,8,8,99,97,3,5,8,44,100,97,
          4,104,2,100,0,1,101,97,5,106>>).
     ```
    
    * _If test case fail with error `libmozjs185.so.1.0 not found`. Copy libmozjs185.so.1.0 to /lib64 as follows_

         ```sh
          sudo cp /usr/lib/libmozjs185.so.1.0 /usr/lib64/libmozjs185.so.1.0
         ```

7. Verify CouchDB by starting the server

    ```
    sudo /<source_root>/couchdb/dev/run &
    ```
 
    curl http://127.0.0.1:15984/ should display
    ```
    {"couchdb":"Welcome","uuid"="xxx","version":"2.0","vendor":{"name":"The Apache Software Foundation"}}
    ```
