#Building CouchDB on SLES 12

CouchDB can be built for a Linux on z System running SLES 12 by following these instructions.

_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified._

1. Firstly, install all of the build-time dependencies(available from the SLES subscription repositories):

        sudo zypper install libicu-devel libcurl4 libcurl-devel java-1_7_1-ibm java-1_7_1-ibm-devel git wget mozilla-nspr mozilla-nspr-devel autoconf213 gcc-c++ zip fontconfig fontconfig-devel pkg-config libasound2 yasm tar make gtk2-devel libXt-devel libIDL-2-0 libidl-devel libfreetype6 libgstreamer-1_0-0 dbus-1-devel dbus-1-glib-devel libnotify4 libnotify-devel python wget perl perl-base
You may already have some of these packages installed - just install any that are missing.

2. Move to the location you wish to use as an installation workspace ( known as <source_root> from this point on ):

        cd /<source_root>/

3. Obtain the SpiderMonkey source code

        wget http://ftp.mozilla.org/pub/mozilla.org/js/js185-1.0.0.tar.gz

4. Untar the archive and change working directory:

        tar zxf js185-1.0.0.tar.gz && cd js-1.8.5/js/src

5. Effect the following source code changes ( _NOTE: The given line numbers account for any previous action on the file._) :
Edit file `/<source_root>/js-1.8.5/js/src/jsval.h`
    Remove line 309 ( content: `jsuword        word;` )
    Add a new line after line number `312` and insert the following string:
        `jsuword        asWord;`
    Add a new line after line number `322` and insert the following string:
        `uint32         padding;`
    Add a new line after line number `346` and insert the following string:
        `uint32         padding;`
    Add a new line after line number `355` and insert the following string:
        `jsuword        asWord;`
Save the file. For more info see http://hg.mozilla.org/mozilla-central/rev/6f2c0dbb88d3
Edit file `/<source_root>/js-1.8.5/js/src/jsvalue.h`
    Remove line 294 ( content: `JS_STATIC_ASSERT(offsetof(jsval_layout, s.payload) == 0);` )
    Replace line 731 ( content: `return &data.s.payload.word;` )
    with the following content:
```c
#if JS_BITS_PER_WORD == 32
return reinterpret_cast<const jsuword *>(&data.s.payload.word);
#elif JS_BITS_PER_WORD == 64
return reinterpret_cast<const jsuword *>(&data.asBits);
#endif ```
Save the file. For more info see https://bugzilla.mozilla.org/attachment.cgi?id=517107&action=diff
Edit file `/<source_root>/js-1.8.5/js/src/Makefile.in`
    Change line number 385:
    From:
    `ifeq (,$(filter-out powerpc sparc,$(TARGET_CPU)))`
    To:
    `ifeq (,$(filter arm %86 x86_64,$(TARGET_CPU)))`
Save the file. For more info see https://bugzilla.mozilla.org/attachment.cgi?id=520157&action=diff
6. Prepare the src code using autoconf

        autoconf-2.13

7. Create a workspace directory and cd into it

        mkdir build_OPT.OBJ && cd build_OPT.OBJ

8. Run the Configure step

        ../configure --prefix=/usr

9. Go ahead and `make`

        make

10. To verify the build, run the tests

        make check

11. If no error is found, install SpiderMonkey:

        sudo make install

12. Now install Erlang according to the instructions in [Building Erlang on SLES12](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang-on-SLES12 "Building Erlang on SLES12").

13. Download ( into `/<source_root>/` ) IBM SDK for Node.js ( version 1.1 ) from developerWorks. Be sure to select the binary for "Linux on System z". Ensure the user you are logged in with has execute permissions on the downloaded binary. Run the downloaded interactive installer and specify `/home/<user_name>/ibm/node/` as the installation directory.

        /<source_root>/node-<VERSION_NUMBER_HERE>-linux-s390x.bin

14. Set the `$PATH` environment variable to include all required directories:

        export PATH=/home/<user_name>/ibm/node/bin:/usr/local/bin:$PATH

15. Install Grunt using NPM:

        npm install -g grunt-cli

16. Building CouchDB requires `rebar`. Build it from source by running:

        git clone git://github.com/rebar/rebar.git
        cd rebar
        ./bootstrap

17. The `bootstrap` command produces an Erlang escript named rebar. Copy it into a directory in your `$PATH`:

        sudo cp /<source_root>/rebar/rebar /usr/local/bin/

18. Now set up the CouchDB source tree:

        cd /<source_root>/
        git clone https://git-wip-us.apache.org/repos/asf/couchdb.git
        cd couchdb
NOTE: If you wish to build the developer preview version of CouchDB, run: git checkout developer-preview-2.0

19. Configure the source tree (which will fetch more source code from version control), then run make to build CouchDB:

        ./configure -p /opt/couchdb -c
        export LD_LIBRARY_PATH=/usr/lib

20. Now make and run some sanity tests:

        make check

21. Verify that the build works by starting the test server (you can kill it with Ctrl-C):

        /<source_root>/couchdb/dev/run

22. Generate an installable release with an embedded Erlang run-time system:

        make dist

23. The /<source_root>/couchdb/rel/ sub-directory should now contain a complete CouchDB installation. It can be installed simply by moving it into place:

        sudo cp -R /<source_root>/couchdb/rel/couchdb /opt/couchdb