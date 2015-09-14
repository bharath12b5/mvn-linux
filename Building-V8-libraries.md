The V8 JavaScript engine can be built for Linux on z Systems running RHEL 7, RHEL 6, SLES 12 or SLES 11 by following these instructions. It can be built as a set of shared libraries so that it can be used by multiple applications.
Versions 3.14 and 3.28 have been successfully built and tested this way. (Note: If you are building V8 for use with [MongoDB](../Building MongoDB), you need V8 3.14.)

More information on the V8 JavaScript engine is available at [the V8 website](https://developers.google.com/v8/intro) and the source code for the z Systems port can be obtained from [GitHub](https://github.com/andrewlow/v8z).

_**General Notes:**_

- When following the steps below, please use a standard permission user unless otherwise specified.

- A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you would like to place it.

## Pre-requisites

1. Issue the following commands to install pre-built dependencies:

    (For RHEL 7.1 and RHEL 6.6)
    ```shell
    sudo yum install git subversion make gcc-c++ tar wget
    ```

    (For SLES 12)
    ```shell
    sudo zypper install git subversion make gcc-c++ tar wget
    ```

    (For SLES 11)
    ```shell
    sudo zypper install git subversion make gcc47-c++ tar wget
    sudo ln -s /usr/bin/gcc-4.7 /usr/bin/gcc
    sudo ln -s /usr/bin/g++-4.7 /usr/bin/g++
    sudo ln -s /usr/bin/gcc /usr/bin/cc
    ```

2. Create a temporary installation directory:

    ```shell
    mkdir /<source_root>
    ```

3. **(V8 3.28 only)** Clone depot_tools from the Chromium project, as it contains a number of tools needed to build V8 3.28:

    ```shell
    cd /<source_root>/
    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
    export PATH=$(pwd)/depot_tools:$PATH
    ```

4. **(V8 3.28 only)** Building V8 3.28 or newer requires Python 2.7, which is not available as a pre-built RPM from RHEL 6 or SLES 11. Before building V8, you need to build Python 2.7, which in turn has a dependency on OpenSSL 1.0.2a on SLES 11. The next two sections describe how to build OpenSSL 1.0.2a and Python 2.7.

   If you are building V8 3.28 on RHEL 7 or SLES 12, Python 2.7 should already be avaialble and V8 3.28 should build correctly, so just skip ahead to [Building V8](#building-v8).

### **(V8 3.28 on SLES 11 only)** Install OpenSSL 1.0.2a

1. Download the OpenSSL source code

    ```shell
    cd /<source_root>/
    wget ftp://ftp.openssl.org/source/openssl-1.0.2a.tar.gz
    tar -xf openssl-1.0.2a.tar.gz
    ```

2. Configure and make:
 
    ```shell
    cd /<source_root>/openssl-1.0.2a
    ./Configure linux64-s390x
    make
    ```

3. Install OpenSSL:

    ```shell
    sudo make install
    export PATH=/usr/local/ssl/bin:$PATH
    ```

4. **(Optional)** Verify the OpenSSL installation; the following command should display the version of OpenSSL that has been built, e.g. "OpenSSL 1.0.2a 19 Mar 2015":

    ```shell
    cd /<source_root>
    openssl version
    ```

### **(V8 3.28 on RHEL 6 or SLES 11)** Install Python 2.7.8

1. Issue the following commands to obtain pre-built dependencies:

    (For RHEL 6)
    ```shell
    sudo yum install ncurses-devel patch zlib-devel bzip2-devel xz openssl-devel
    ```

    (For SLES 11)
    ```shell
    sudo zypper install ncurses-devel patch zlib-devel libbz2-devel
    ```
  
2. Download the Python source code:

    ```shell
    cd /<source_root>/
    wget https://www.python.org/ftp/python/2.7.8/Python-2.7.8.tar.xz
    tar -xf Python-2.7.8.tar.xz
    ```

3. Configure:

    ```shell
    cd /<source_root>/Python-2.7.8
    ./configure --prefix=/usr/local --exec-prefix=/usr/local
    ```

4.  **(SLES 11 only)** Edit Modules/Setup to replace the following block:

    ```shell
    #SSL=/usr/local/ssl
    #_ssl _ssl.c \
    #       -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
    #       -L$(SSL)/lib -lssl -lcrypto
    ```

    with the following:

    ```shell
    SSL=/usr/local/ssl
    _ssl _ssl.c \
        -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
        $(SSL)/lib/libssl.a $(SSL)/lib/libcrypto.a
    #   -L$(SSL)/lib -lssl -lcrypto
    _hashlib _hashopenssl.c \
        -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
        $(SSL)/lib/libssl.a $(SSL)/lib/libcrypto.a
    ```

   Note that the indented lines 3, 4, 7 and 8 must begin with a single tab character (not spaces).

5. Make and install:

    ```shell
    make
    sudo make install
    ```

6. **(Optional)** Verify the Python installation; the following command should display the version of Python that has been built, i.e. "Python 2.7.8":

    ```shell
    cd /<source_root>
    python --version
    ```

### **(V8 3.28 on RHEL 6 only)** Install git 2.2.1    

depot-tools requires a later version of git than that which is available through yum install. This can be achieved by building a suitable version of git from source, as follows:

1. Issue the following command to obtain additional pre-built dependencies:

    ```shell
    sudo yum install autoconf perl-devel
    ```

2. Download the git source code:

    ```shell
    cd /<source_root>/
    wget https://github.com/git/git/archive/v2.2.1.tar.gz
    tar -xf v2.2.1.tar.gz
    ```

3. Configure and make:
 
    ```shell
    cd /<source_root>/git-2.2.1
    make configure
    ./configure --prefix=/usr --without-tcltk
    make NO_GETTEXT=1
    ```

4. Install:

    ```shell
    sudo make install NO_GETTEXT=1
    ```

5. **(Optional)** Verify the git installation; the following command should display the version of git that has been built, i.e. "git version 2.2.1":

    ```shell
    cd /<source_root>
    git --version
    ```

## Building V8

1. Obtain the V8 source code:

    ```shell
    cd /<source_root>/
    git clone https://github.com/andrewlow/v8z.git
    cd v8z
    ```

2. Select the version to be built:

   (For 3.28)
    
    ```shell
    git checkout 3.28-s390
    ```

   (For 3.14)
    
    ```shell
    git checkout 3.14-s390
    ```

3. Fetch source code for dependent libraries:

    ```shell
    make dependencies
    ```

4. **(SLES 11 only)** Issue the following command:

    ```shell
    sed -i "s/crsT/crs/g" build/gyp/pylib/gyp/generator/make.py
    ```

   This is needed in order to prevent the `ar` command from failing when creating libraries during the build process.

6. Build the shared library:

    ```shell
    make s390x -j4 library=shared
    ```

   The 64-bit shared library will be output as `v8z/out/s390x.release/lib.target/libv8.so`. When building V8 3.28, libicui18n.so and libicuuc.so are also created in the same directory as libv8.so.

7. V8 header files will be required to build products invoking the V8 APIs. Issue the following commands to install the header files:

    ```shell
    cd /<source_root>/v8z
    sudo cp -vR include/* /usr/include/
    sudo chmod -f 644 /usr/include/v8*h /usr/include/libplatform/libplatform.h
    ```
  
7. Install the V8 libraries into /usr/local/lib64/ (or /usr/lib64/ if you prefer):

    ```shell
    cd /<source_root>/v8z
    
    sudo cp -v out/s390x.release/lib.target/lib*.so /usr/local/lib64/
    sudo chmod -f 755 /usr/local/lib64/libv8.so /usr/local/lib64/libicu*.so
    
    sudo cp -v out/s390x.release/obj.target/tools/gyp/lib*.a /usr/local/lib64/
    if [ -d out/s390x.release/obj.target/third_party/icu ] ; then \
        sudo cp -v out/s390x.release/obj.target/third_party/icu/lib*.a /usr/local/lib64/ ; fi
    sudo chmod -f 644 /usr/local/lib64/libv8*.a /usr/local/lib64/libpreparser_lib.a /usr/local/lib64/libicu*.a

    sudo ldconfig -v
    ```

   Note that the above `cp` commands use the `-v` option in order to list the files that are being installed.

8. **(Optional)** Check that the sample shell can be invoked and that it displays the expected V8 version. To exit from the shell, enter `quit()`.

    ```shell
    /<source_root>/v8z/out/s390x.release/shell
    ```

9. **(Optional)** Once you have installed the V8 libraries and header files outside `/<source_root>/` then `/<source_root>/` may be deleted.

## References

- [Official website for the V8 JavaScript engine](https://developers.google.com/v8/intro)
- [More information on building the V8 JavaScript engine](https://code.google.com/p/v8/)
