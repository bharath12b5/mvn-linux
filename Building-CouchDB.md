#Building [CouchDB](https://couchdb.apache.org/) on SLES ( 11.3 & 12.0 ) and RHEL ( 6.6 & 7.1 )

CouchDB can be built for a Linux on z System running SLES 11.3/12.0 & RHEL 6.6/7.1 by following these instructions.

_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified._

1. First, install the build-time dependencies:

    On SLES 11.3:

        sudo zypper install libicu-devel libcurl4 libcurl-devel git wget mozilla-nspr mozilla-nspr-devel autoconf213 zip fontconfig fontconfig-devel pkg-config libasound2 yasm tar make gtk2-devel libXt-devel libIDL-2-0 libidl-devel libfreetype6 libgstreamer-1_0-0 dbus-1-devel dbus-1-glib-devel libnotify4 libnotify-devel python wget perl perl-base java-1_7_0-ibm java-1_7_0-ibm-devel libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel

    On SLES 12.0:

        sudo zypper install libicu-devel libcurl4 libcurl-devel git wget mozilla-nspr mozilla-nspr-devel autoconf213 zip fontconfig fontconfig-devel pkg-config libasound2 yasm tar make gtk2-devel libXt-devel libIDL-2-0 libidl-devel libfreetype6 libgstreamer-1_0-0 dbus-1-devel dbus-1-glib-devel libnotify4 libnotify-devel python wget perl perl-base java-1_7_1-ibm java-1_7_1-ibm-devel libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel

    On RHEL 6.6:

    _**NOTE:** As root, add the RHEL 6 Server Optional repository to the yum repository list. This is required to get the IBM Java package. See the instructions in [Adding RHEL Optional and Supplementary Repositories](https://github.com/linux-on-ibm-z/docs/wiki/Adding-RHEL-Optional-and-Supplementary-Repositories)._

        sudo yum install libicu-devel libcurl libcurl-devel git wget zip bzip2 tar nspr nspr-devel autoconf213 fontconfig fontconfig-devel pkgconfig make gtk2-devel libXt-devel libIDL libIDL-devel freetype freetype-devel gstreamer gstreamer-devel dbus dbus-devel dbus-glib dbus-glib-devel libnotify libnotify-devel python perl java-1.7.1-ibm java-1.7.1-ibm-devel

    On RHEL 7.1:

    _**NOTE:** As root, add the RHEL 7 Server Optional repository to the yum repository list. This is required to get the IBM Java package. See the instructions in [Adding RHEL Optional and Supplementary Repositories](https://github.com/linux-on-ibm-z/docs/wiki/Adding-RHEL-Optional-and-Supplementary-Repositories)._

        sudo yum install libicu-devel libcurl libcurl-devel git wget zip bzip2 tar nspr nspr-devel autoconf213 fontconfig fontconfig-devel pkgconfig make gtk2-devel libXt-devel libIDL libIDL-devel freetype freetype-devel gstreamer1 gstreamer1-devel dbus dbus-devel dbus-glib dbus-glib-devel libnotify libnotify-devel python perl java-1.7.1-ibm java-1.7.1-ibm-devel

2. Now install gcc-c++

    On SLES 11.3:

        sudo zypper install gcc47-c++
        sudo ln -s /usr/bin/gcc-4.7 /usr/bin/gcc
        sudo ln -s /usr/bin/gcc-4.7 /usr/bin/cc
        sudo ln -s /usr/bin/g++-4.7 /usr/bin/g++
        sudo ln -s /usr/bin/g++-4.7 /usr/bin/c++

    On SLES 12.0:

        sudo zypper install gcc-c++

    On RHEL 6.6 & 7.1:

        sudo yum install gcc-c++

3. Move to the location you wish to use as an installation workspace ( known as `<source_root>` from this point on )

        cd /<source_root>/

4. Obtain the [SpiderMonkey](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Releases/1.8.5) source code

        wget http://ftp.mozilla.org/pub/mozilla.org/js/js185-1.0.0.tar.gz

5. Untar the archive and change working directory:

        tar zxf js185-1.0.0.tar.gz && cd js-1.8.5/js/src

6. Effect the following source code changes
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

7. Prepare the src code using autoconf

        autoconf-2.13

8. Create a workspace directory and cd into it

        mkdir /<source_root>/js-1.8.5/js/src/build_OPT.OBJ
        cd /<source_root>/js-1.8.5/js/src/build_OPT.OBJ

9. Run the Configure step

        ../configure --prefix=/usr

10. Go ahead and make SpiderMonkey

        make

11. To verify the SpiderMonkey build, run the tests

        make check

12. If no error is found, install SpiderMonkey:

        sudo make install

13. Install Erlang:
  
  _**NOTE:** Steps 1 & 2 within the following Erlang instruction links should be ignored ( as we have already covered those steps earlier in this recipe )._

      cd /<source_root>/
  - For SLES 11.3 & 12.0, build & install Erlang according to the instructions in [Building Erlang on SLES12](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang-on-SLES12 "Building Erlang on SLES12").
  - For RHEL 6.6 & 7.1, build & install Erlang according to the instructions in [Building Erlang on RHEL7](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang-on-RHEL7 "Building Erlang on RHEL7").

14. Download ( into `/<source_root>/` ) IBM SDK for Node.js ( version 1.1 ) from [developerWorks](http://www.ibm.com/developerworks/web/nodesdk/). Be sure to select the binary for "Linux on System z". Ensure the user you are logged in with has execute permissions on the downloaded binary. Run the downloaded interactive installer and specify `/home/<user_name>/ibm/node/` as the installation directory.

        /<source_root>/node-<VERSION_NUMBER_HERE>-linux-s390x.bin

15. Set the PATH environment variable to include all required directories:

        export PATH=/home/<user_name>/ibm/node/bin:/usr/local/bin:$PATH

16. Install Grunt using NPM:

        npm install -g grunt-cli

17. Building CouchDB requires `rebar`. Build it from source by running:

        git clone git://github.com/rebar/rebar.git
        cd rebar
        ./bootstrap

18. The bootstrap command produces an Erlang escript named rebar. Copy it into a directory in your `$PATH`:

        sudo cp /<source_root>/rebar/rebar /usr/local/bin/

19. Now set up the CouchDB source tree:

        cd /<source_root>/
        git clone https://git-wip-us.apache.org/repos/asf/couchdb.git
        cd couchdb
    _**NOTE:** If you wish to build the developer preview version of CouchDB, run: `git checkout developer-preview-2.0`_

20. Configure the source tree (which will fetch more source code from version control), then run make to build CouchDB:

        ./configure -p /opt/couchdb -c
        export LD_LIBRARY_PATH=/usr/lib

21. Now make and run some sanity tests.

        make check

22. Verify that the build works by starting the test server (you can kill it with Ctrl-C):

        /<source_root>/couchdb/dev/run

23. Generate an installable release with an embedded Erlang run-time system:

        make dist

24. The `/<source_root>/couchdb/rel/` sub-directory should now contain a complete CouchDB installation. It can be installed simply by moving it into place:

        sudo cp -R /<source_root>/couchdb/rel/couchdb /opt/couchdb
