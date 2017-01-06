.<!---PACKAGE:CouchDB--->
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

The instructions provided below specify the steps to build CouchDB Version 2.0.0 on IBM z Systems for RHEL 7.1/7.2, SLES 12-SP1 and Ubuntu16.04.

### _**General Note:**_
i)  _When following the steps below please use a standard permission user unless otherwise specified._

### Prerequisites
  * NodeJS v6.9.0(SLES 12-SP1, RHEL 7.1/7.2 & Ubuntu 16.04)
  -- Download NodeJS v6.9.2 binary from [NodeJS 6.9.0](https://developer.ibm.com/node/sdk/#v6) and follow the instructions as per given in the link
  * PhantomJS (RHEL 7.1/7.2 & SLES 12-SP1)
  -- Instructions for building PhantomJS can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-PhantomJS)
  

### Building and Installing CouchDB
1. First, install the build dependencies


    RHEL 7.1/7.2


        sudo yum install libicu-devel libcurl libcurl-devel git wget zip bzip2 tar nspr nspr-devel autoconf213 fontconfig fontconfig-devel pkgconfig make gtk2-devel libXt-devel libIDL libIDL-devel freetype freetype-devel gstreamer1 gstreamer1-devel dbus dbus-devel dbus-glib dbus-glib-devel libnotify libnotify-devel python perl java-1.7.1-ibm java-1.7.1-ibm-devel xz libtool which texlive.s390x texlive-titlesec.noarch texlive-framed.noarch texlive-threeparttable.noarch texlive-wrapfig.noarch texlive-multirow.noarch texinfo.s390x curl



    SLES 12-SP1

        sudo zypper install libicu-devel libcurl4 libcurl-devel git wget mozilla-nspr mozilla-nspr-devel autoconf213 zip fontconfig fontconfig-devel pkg-config libasound2 yasm tar make gtk2-devel libXt-devel libIDL-2-0 libidl-devel libfreetype6 libgstreamer-1_0-0 dbus-1-devel dbus-1-glib-devel libnotify4 libnotify-devel python wget perl perl-base java-1_7_1-ibm java-1_7_1-ibm-devel libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel libtool pkg-config curl


    Ubuntu 16.04

        sudo apt-get update
        sudo apt-get install -y openjdk-8-jdk texlive-full ncurses-base gcc python git wget tar make autoconf2.13 automake autoconf g++ libmozjs185-1.0 libmozjs185-dev  openssl  libicu57  libicu-dev libcurl4-openssl-dev  build-essential unixodbc libncurses5-dev libncursesw5-dev fop libncurses5-dev unixodbc-dev libssl-dev pkg-config curl phantomjs


2. Now install gcc-c++

    
    On RHEL 7.1/7.2:
    
    ```
    sudo yum install gcc-c++
    ```
    

    On SLES 12-SP1:
    
    ```
    sudo zypper install gcc-c++
    ```



3. Install SpiderMonkey

    i) Obtain the [SpiderMonkey](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Releases/1.8.5) source code
    ```
    cd /<source_root>/
    wget http://ftp.mozilla.org/pub/mozilla.org/js/js185-1.0.0.tar.gz
    ```
    
    ii) Untar the archive and change working directory:
    ```
    tar zxf js185-1.0.0.tar.gz && cd js-1.8.5/js/src
    ```

    iii) Effect the following source code changes

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

    iv) Prepare the src code using autoconf

        autoconf-2.13

    v) Create a workspace directory and cd into it

        mkdir /<source_root>/js-1.8.5/js/src/build_OPT.OBJ
        cd /<source_root>/js-1.8.5/js/src/build_OPT.OBJ

    vi) Run the Configure step

        ../configure --prefix=/usr

    vii) Go ahead and make SpiderMonkey

        make

    viii) If no error is found, install SpiderMonkey:

        sudo make install

4. Install Erlang
  
  _**NOTE:** Steps 1 & 2 within the following Erlang instruction links should be ignored ( as we have already covered those steps earlier in this recipe )._

      cd /<source_root>/
  - For 12-SP1, build & install Erlang according to the instructions in [Building Erlang on SLES12](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang-on-SLES12 "Building Erlang on SLES12").
  - For 7.1/7.2, build & install Erlang according to the instructions in [Building Erlang on RHEL7](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang-on-RHEL7 "Building Erlang on RHEL7").

  - For Ubuntu
  ```
	wget http://www.erlang.org/download/otp_src_17.4.tar.gz
	tar zxvf otp_src_17.4.tar.gz
	cd otp_src_17.4
	export ERL_TOP=`pwd`
	./configure --prefix=/usr
	make
	sudo make install
    ```

5. Build and install Rebar

     * CouchDB requires `rebar`, build it from source by running:

            cd /<source_root>/
            git clone git://github.com/rebar/rebar.git
            cd /<source_root>/rebar
            ./bootstrap

    * The bootstrap command produces an Erlang escript named rebar. Copy it into a directory in your `$PATH`:

            sudo cp /<source_root>/rebar/rebar /usr/local/bin/
		
6. Install Autoconf-archive

        cd /<source_root>/
        wget http://infinity.kmeacollege.ac.in/gnu/autoconf-archive/autoconf-archive-2016.03.20.tar.xz
        tar -xvf autoconf-archive-2016.03.20.tar.xz
		cd autoconf-archive-2016.03.20
		./configure
		make
		sudo make install
		sudo cp ./m4/* /usr/share/aclocal (Only for RHEL / Ubuntu)
		

7. Install "prove" module using CPAN (Only for RHEL6.7)
    ````
    /usr/bin/perl -MCPAN -e 'prove'
    ````
		
8. Now set up the CouchDB source tree

        cd /<source_root>/
        git clone https://git-wip-us.apache.org/repos/asf/couchdb.git
        cd /<source_root>/couchdb
        git checkout 2.0.0

9. Configure the source tree (which will fetch more source code from version control), then run make to build CouchDB
     ```
    ./configure -c --disable-docs --disable-fauxton
    export LD_LIBRARY_PATH=/usr/lib
    make		
    ```
10. Installing CouchDB

    ``` 
    sudo make install
     ```   
11. Now make and run some sanity tests

     ````
    make check
     ````

   _**Note:** Three test case failures from module `test/couch_compress_tests.erl` at Line 35,49 & 51 are observed and investigation is in progress_


13. Verify CouchDB by starting the server

    ```
    sudo /<source_root>/couchdb/dev/run &
    ```
 
    curl http://127.0.0.1:5984/ should display
    ```
    {"couchdb":"Welcome","uuid"="xxx","version":"2.0","vendor":{"name":"The Apache Software Foundation"}}
    ```
