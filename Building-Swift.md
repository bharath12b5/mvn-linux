Swift is available on RHEL 7.1, SLES 12.1 and Ubuntu 16.04. Swift 3.0 has been built and tested on Linux on z Systems.
This is a Beta release to go in hand with the official Swift 3.0 release.

### Preparing to build LLVM/clang

1. Install prerequisites:

    For RHEL 7.1

        sudo yum install binutils-devel gcc-c++ git \
                         libcurl-devel bzip2-devel sqlite3-devel libbsd \
                         libicu-devel libuuid-devel libxml2-devel ncurses-devel \
                         python-devel swig xz-devel

        wget https://cmake.org/files/v3.5/cmake-3.5.2.tar.gz
        tar xzf cmake-3.5.2.tar.gz
        cd cmake-3.5.2
        ./configure --prefix=/opt/cmake
        make
        make install
        export PATH=$PATH:/opt/cmake/bin      

        cd ..
        git clone https://github.com/ninja-build/ninja.git && cd ninja
        git checkout release      
        /configure.py --bootstrap
        export PATH=$PATH:`pwd`
        

    For SLES 12

        sudo zypper install binutils-devel gcc-c++ git \
                            libarchive-devel libbz2-devel libcurl-devel \
                            libedit-devel libexpat-devel libicu-devel \
                            libuuid-devel libxml2-devel ncurses-devel \
                            pkg-config python-devel sqlite3-devel swig xz-devel

        rpmbuild --rebuild cmake-3.4.3-237.1.src.rpm 
        rpmbuild --rebuild ninja-1.5.1-1.1.src.rpm
        rpmbuild --rebuild libbsd-0.7.0-2.1.src.rpm

        cd $HOME/rpmbuild/RPMS/s390x/
        sudo rpm -i --force cmake-3.4.3-237.1.s390x.rpm
        sudo rpm -i libbsd0-0.7.0-2.1.s390x.rpm \
                    libbsd-devel-0.7.0-2.1.s390x.rpm \
                    ninja-1.5.1-1.1.s390x.rpm
                    
        cd /usr/lib64
        sudo ln -s libncurses.so libcurses.so

        cd /usr/include
        sudo ln -s /usr/include/ncurses/panel.h

				
    For Ubuntu 16.04

        sudo apt-get install autoconf libtool git cmake ninja-build python \
                             python-dev python3-dev uuid-dev libicu-dev \
                             icu-devtools libbsd-dev libedit-dev libxml2-dev \
                             libsqlite3-dev swig libpython-dev libncurses5-dev \
                             pkg-config libcurl4-nss-dev
       
2.  Install other prerequisites from Github:

        git clone https://github.com/mackyle/blocksruntime
        cd blocksruntime
        ./buildlib
        sudo ./installlib
      
        git clone https://github.com/mheily/libkqueue
        cd libkqueue
        autoreconf -i
        ./configure
        make
        sudo make install

3. You will also need the gold linker (part of the binutils package), which was ported to z in late 2015.  See [[Building Gold Linker]] for details on building binutils from source and installing it. Then, add the binutils installation directory to the PATH variable:

        export PATH=/opt/binutils-2.26/bin:$PATH

4. (For SLES 12 only) If your system has libstdc++ 6.0.21 but older GCC/binutils (< GCC 5), then you should downgrade libstdc++ to work around this issue:

    https://bugs.swift.org/browse/SR-23

   A copy of libstdc++ 6.0.19 can be copied from a SLES 12 (< SP1) system and symlinked:

        cd /usr/lib64
        rm libstdc++.so.6
        ln -s libstdc++.so.6.0.19 libstdc++.so.6

   The version of this package can be locked to prevent accidental upgrades:

        zypper addlock libstdc++6-4.8.3+r212056-6.3

### Building LLVM/clang

LLVM 3.9 or above is needed for building Swift and its components.

1. Clone the LLVM and clang code:

    Use the steps here to obtain the version tested, available in linux-on-ibm-z GitHub.
     
        git clone https://github.com/linux-on-ibm-z/llvm.git
        cd llvm
        git checkout llvm-for-swift-3.0
        cd tools
        git clone https://github.com/linux-on-ibm-z/clang.git
        cd clang
        git checkout clang-for-swift-3.0
        cd ../../projects
        git clone https://github.com/linux-on-ibm-z/compiler-rt.git
        cd compiler-rt
        git checkout compiler-rt-for-swift-3.0 && cd ..

    Alternatively, you can try the official current trunk
    
        git clone http://llvm.org/git/llvm.git
        cd llvm/tools
        git clone http://llvm.org/git/clang.git
        git clone http://llvm.org/git/lldb.git  # OPTIONAL
        cd ../projects
        git clone http://llvm.org/git/compiler-rt.git
   
2. Run cmake outside the source tree to configure LLVM:

    For SLES 12 & RHEL 7.1

        cd ../..
        mkdir build

        cd build
        cmake -DCMAKE_BUILD_TYPE=Release \
              -DCMAKE_INSTALL_PREFIX=/opt/llvm \
              ../llvm
        make -j4
        make check   # optional
        make install # or "sudo make install"

    For Ubuntu 16.04  (Library discovery on Ubuntu 16.04 seems to be very broken, hence the extra -D options, in order to allow LLDB to build):
    
        cd ../..
        mkdir build

        cd build
        cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/opt/llvm \
              -DPYTHON_INCLUDE_DIR=/usr/include/python2.7 \
              -DPYTHON_LIBRARY=/usr/lib/python2.7/config-s390x-linux-gnu/libpython2.7.so \
              -DCURSES_INCLUDE_PATH=/usr/include \
              -DCURSES_LIBRARY=/usr/lib/s390x-linux-gnu/libncurses.so \
              -DCURSES_PANEL_LIBRARY=/usr/lib/s390x-linux-gnu/libpanel.so \
              ../llvm
        make -j4
        make check   # optional
        make install # or "sudo make install"


    The build is highly parallelizable, try not to use more than 4 job slots when building LLVM with pararllel make, the machine will quickly run out of memory, and everything will slow to a crawl or die randomly.

3. Add the new LLVM installation to the PATH variable:

        export PATH=/opt/llvm/bin:$PATH


### Building Swift

1. Clone the from the official repository:

        git clone https://github.com/apple/swift.git
        cd swift
        git checkout swift-3.0-branch

    or from our team internal github site (which may have additional fixes pending upstream)

        git clone git@github.com:linux-on-ibm-z/swift.git
        cd swift
        git checkout swift-3.0-branch
   
        
2. Run the update-checkout script to clone everything else from upstream (for now):

    For SLES 12 & RHEL 7.1

        ./utils/update-checkout --clone-with-ssh --branch swift-3.0-branch
        

    For Ubuntu 16.04

        ./utils/update-checkout --clone --branch swift-3.0-branch


3. Apply this patch https://github.com/apple/swift-llvm/pull/21 if it has not been merged.  It was not picked up in time for swift-3.0-branch and when this recipe is published.

    `cd ../llvm` to apply the patch and move back to the swift directory with `cd ../swift` when done.

4. Build the code (`$DESTDIR` can be any blank directory where the installable package is assembled, for example `$HOME/swift/install`):

    For SLES 12 & RHEL 7.1

        ./utils/build-script -j 2 -r \
        --lldb --foundation --xctest --llbuild --swiftpm --libdispatch -- \
        --verbose-build=1 \
        --install-swift --install-foundation --install-xctest --install-llbuild --install-swiftpm --install-libdispatch \
        --swift-install-components='autolink-driver;compiler;clang-builtin-headers;stdlib;sdk-overlay;license' \
        --build-swift-static-stdlib=1 \
        --install-prefix=/usr \
        --install-destdir=$DESTDIR

    For Ubuntu 16.04

    Currently, due to the above-mentioned CMake library discovery problem, LLDB/REPL cannot be built.     

        ./utils/build-script -j 2 -r -- \
        --foundation --xctest --llbuild --swiftpm --libdispatch -- \
        --verbose-build=1 \
        --install-swift --install-foundation --install-xctest --install-llbuild --install-swiftpm --install-libdispatch \
        --swift-install-components='autolink-driver;compiler;clang-builtin-headers;stdlib;sdk-overlay;license' \
        --build-swift-static-stdlib=1 \
        --install-prefix=/usr \
        --install-destdir=$DESTDIR

   _Notes:_ 
   
    1. The -j option must be set to a small number of parallel build threads (e.g. 2), otherwise the linkers will exhaust all system memory during the build.
  
    2. --install-prefix must either be omitted or set to its default value of "/usr".
    
    3. When the build completes, the Swift binaries will be laid out under $DESTDIR, with the complete prefix (i.e. usr). The contents of the installation destination can be moved to another location (e.g. /opt/swift), or tarred up and unpacked on another machine.

    4. Additional useful build options

        - Add the -c (--clean) option before the "--" delimiter to start the build from scratch; i.e. do not attempt an incremental build.

        - Replace the -r option with -d to get a debug build.
 
        - Replace the -r option with -R to get a release build (no debugging symbols and no assert).

    
    _Additional Notes **(For SLES 12 & RHEL 7.1)**:_ 
    
    Currently, the build will stop during the installation phase due to a known issue in LLDB's CMake files.

    This is a LLDB bug (https://llvm.org/bugs/show_bug.cgi?id=23785), preventing the installation of REPL/LLDB on SLES and RHEL:
    
        CMake Error at scripts/cmake_install.cmake:36 (file):
          file INSTALL cannot find
          "/localbox/bryanpkc/swift/build/Ninja-RelWithDebInfoAssert/lldb-linux-s390x/lib/python2.7".
        Call Stack (most recent call first):
          cmake_install.cmake:42 (include)
    
    According to SR-39 (https://bugs.swift.org/browse/SR-39), this can be worked around by adjusting the generated Ninja scripts to use lib64/python2.7 instead of lib/python2.7. Two files need to be changed:
    
    - build/Ninja-RelWithDebInfoAssert/lldb-linux-s390x/scripts/cmake_install.cmake
    
       Replace all occurrences of /lib with /lib64, i.e. (in vim)
    
       :%s#/lib#/lib64#g
    
    - build/Ninja-RelWithDebInfoAssert/lldb-linux-s390x/scripts/Python/modules/readline/cmake_install.cmake
    
       Replace all occurrences of lib/python2.7 with lib64/python2.7, i.e.
    
       :%s#/lib/python2.7#/lib64/python2.7#g
    
    After editing the cmake_install.cmake files as described, the (incremental, i.e. no -c) build-script command can be re-issued and it will pick from where it left off and finish the installation. The corrected files will continue to work for future incremental builds unless the build is reconfigured with CMake.



### Using Swift

To run the Swift compiler:

    $ export PATH=$DESTDIR/opt/swift/bin:/opt/llvm/bin:/opt/binutils-2.26/bin:$PATH

    $ swiftc -v hello.swift

    $ ./hello
    Hello World

If the LLVM and Clang binaries are not found on $PATH, swiftc will fail in the link step with this cryptic message:

    <unknown>:0: error: link command failed with exit code 127 (use -v to see invocation)

To run the symbol demangler:

    $ swift-demangle _TSS
    _TSS ---> Swift.String

#### Useful Swift Options

1. To make the Swift frontend generate Swift IL for a .swift program, add the option -emit-sil.

2. To make the Swift frontend generate LLVM IR for a .swift program, add the option -emit-ir. The LLVM IR output can be fed to llc to debug optimization and code generation.

The output of -emit-sil and -emit-ir can be filtered through swift-demangle to get human-readable function names. But the filtered LLVM IR may be unusable by llc.


### Running Test

To run unit tests

    ./utils/build-script -j 2 -r -t

And for the complete validation test suite

    ./utils/build-script -j 2 -r -T

It is possible to add the -t or -T option to the build Swift command to build and test at the same run.

Currently, validation test pass rate is at 99.8% and there are 13 test failures.

### Swift LinuxONE Sandbox

IBM Swift Sandbox (https://swiftlang.ng.bluemix.net/#/repl) also supports running Swift code in LoZ.  

To run Swift code in Swift LinuxONE Sandbox


* Click on "Settings" in the bottom right corner, select LinuxONE as the architecture

* Put or load Swift code in the left pane
   
* Click the Play icon in the bottom to see the code at work

