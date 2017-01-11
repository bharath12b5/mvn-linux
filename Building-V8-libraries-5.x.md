# Building V8 5.6

The instructions provided below specify the steps to build V8 JavaScript engine Version 5.6 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10. It can be built as a set of shared libraries so that it can be used by multiple applications.
(Note: If you are building V8 for use with [MongoDB](../Building MongoDB), you need V8 3.14.)

More information on the V8 JavaScript engine is available at [the V8 website](https://developers.google.com/v8/intro)

_**General Notes:**_

 * When following the steps below, please use a standard permission user unless otherwise specified.

 * A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you would like to place it.

## Pre-requisites

1. Issue the following commands to install pre-built dependencies

    RHEL 6.8, RHEL 7.1/7.2/7.3
    ```shell
    sudo yum install git subversion make gcc-c++ tar wget which bzip2
    ```

	SLES 11-SP4
    ```shell
	sudo zypper install git subversion make gcc47-c++ tar wget bzip2
    sudo ln -s /usr/bin/gcc-4.7 /usr/bin/gcc
    sudo ln -s /usr/bin/g++-4.7 /usr/bin/g++
    sudo ln -s /usr/bin/gcc /usr/bin/cc
    ```
	
    SLES 12/12-SP1/12-SP2
    ```shell
    sudo zypper install git subversion make gcc-c++ tar wget python-xml
    ```

    Ubuntu 16.04/16.10
    ```shell
	sudo apt-get update
    sudo apt-get install git subversion make tar wget gcc g++ python
    ```    

2. Create a temporary installation directory

    ```shell
    mkdir /<source_root>
    ```

3. Clone depot_tools from the Chromium project, as it contains a number of tools needed to build V8 5.6

    ```shell
    cd /<source_root>/
    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
    export PATH=$(pwd)/depot_tools:$PATH
    ``` 

### Install OpenSSL 1.0.2 **(For SLES 11-SP4  only)** 

1. Download the OpenSSL source code

    ```shell
    cd /<source_root>/
    wget ftp://ftp.openssl.org/source/old/1.0.2/openssl-1.0.2.tar.gz
    tar -xf openssl-1.0.2.tar.gz
    ```

2. Build and install OpenSSL
 
    ```shell
    cd /<source_root>/openssl-1.0.2
    ./Configure linux64-s390x
    make
    sudo make install
    export PATH=/usr/local/ssl/bin:$PATH
    ```

3. **(Optional)** Verify the OpenSSL installations 

    The below command should display the version of OpenSSL that has been built, e.g. "OpenSSL 1.0.2"  

    ```shell
    openssl version
    ```

### Install Python 2.7.12 **(For RHEL 6.8, RHEL 7.1 and SLES 11-SP4 only)** 

1. Issue the following commands to obtain pre-built dependencies

    RHEL 6.8, RHEL 7.1
    ```shell
    sudo yum install ncurses-devel patch zlib-devel bzip2-devel xz openssl-devel
    ```

    SLES 11-SP4
    ```shell
    sudo zypper install ncurses-devel patch zlib-devel libbz2-devel awk xz
    ```
  
2. Download the Python source code

    ```shell
    cd /<source_root>/
    wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tar.xz
    tar -xvf Python-2.7.12.tar.xz
    ```

3. Configure Python

    ```shell
    cd /<source_root>/Python-2.7.12
    ./configure --prefix=/usr/local --exec-prefix=/usr/local
    ```

4.  Edit Modules/Setup to replace the following block **(For SLES 11-SP4 only)** 

    ```shell
    #SSL=/usr/local/ssl
    #_ssl _ssl.c \
    #       -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
    #       -L$(SSL)/lib -lssl -lcrypto
    ```

    with the following

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

5. Build and install Python

    ```shell
    make
    sudo make install
    ```

6. **(Optional)** Verify the Python installation

	The below command should display the version of Python that has been built, i.e. "Python 2.7.12"

    ```shell
    python --version
    ```

### Install git 2.2.1 **(For RHEL 6.8 and 7.1 only)**     
depot-tools requires a later version of git than that which is available through yum install. This can be achieved by  building a suitable version of git from source, as follows:

1. Issue the following command to obtain additional pre-built dependencies

    ```shell
    sudo yum install autoconf perl-devel zlib-devel
    ```

2. Download the git source code

    ```shell
    cd /<source_root>/
    wget https://github.com/git/git/archive/v2.2.1.tar.gz
    tar -xf v2.2.1.tar.gz
    ```

3. Build and install git
 
    ```shell
    cd /<source_root>/git-2.2.1
    make configure
    ./configure --prefix=/usr --without-tcltk
    make NO_GETTEXT=1
    sudo make install NO_GETTEXT=1
    ```
4. **(Optional)** Verify the git installation 

    The below command should display the version of git that has been built, i.e. "git version 2.2.1"   

    ```shell
    git --version
    ```

### Build Gcc 4.8.2 directly **(For RHEL 6.8 and SLES 11-SP4)**

_**Note:** The gcc version available on the RHEL 6.8 and SLES 11-SP4 package repositories is a lower version. It needs to be at least at version 4.8.2, so build and install gcc yourself by following these steps._
	  
1. Download and extract gcc
		
    ```shell
    cd /<source_root>/
    wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.bz2
    bunzip2 gcc-4.8.2.tar.bz2
    tar xvf gcc-4.8.2.tar
    cd gcc-4.8.2/
    ```
2. Automatically download all missing prerequisites

    ```shell
    ./contrib/download_prerequisites
    ```
3. Create a build directory and configure gcc to build

    ```shell
    cd /<source_root>/
    mkdir gccbuild
    cd gccbuild/
    ../gcc-4.8.2/configure --prefix=$HOME/install/gcc-4.8.2 --enable-shared --disable-multilib --enable-threads=posix --with-system-zlib --enable-languages=c,c++
    ```
   _**Note:** You can change the `--prefix=$HOME/install/gcc-4.8.2` directory to be a different installation location if you prefer, however please note the path entry will need changing later_

4. Build and install gcc

    ```shell
    make
    sudo make install
    ```
5. Update the path to match the new gcc version

    ```shell
    export PATH=$HOME/install/gcc-4.8.2/bin:$PATH
    ```
    _**Note:** If you changed the directory in `configure`, ensure it matches above._
	
## Building V8

1. Obtain the V8 source code

    ```shell
	cd /<source_root>/
    fetch v8
	cd v8/
	git pull origin
    ```
	
	_**Note:** (For SLES 11-SP4 Only) If you get the below error while running 'fetch v8'. Ignore it and continue with the recipe if a folder 'v8' already exists._
	```
	Error: Command 'download_from_google_storage --no_resume --platform=linux* --no_auth --bucket chromium-clang-format -s v8/buildtools/linux64/clang-format.sha1'
	```

2. Select the version to be built

    ```shell
    git checkout 5.6.331
    ```
 
3. Download cacert file **(For SLES11-SP4 only)**  

    ```shell
    wget http://curl.haxx.se/ca/cacert.pem --no-check-certificate
    cp cacert.pem cacert.txt
    export SSL_CERT_FILE=/<src_root>/v8/cacert.txt
    ```

4. Fetch source code for dependent libraries

    ```shell
    gclient sync
    ```
	_**Note:**_ If the above step ends prompts you to run 'python gypfiles/gyp_v8', please go ahead and execute the same.

5. Edit make.py file and replace crsT flags with crs using below sed command **(For SLES 11-SP4 only)** 

    ```shell
    sed -i "s/crsT/crs/g" /<src_root>/v8/tools/gyp/pylib/gyp/generator/make.py
    ```

   This is needed in order to prevent the `ar` command from failing when creating libraries during the build process.

6. Build V8 Library
    ```shell

    make s390x -j4 werror=no
    ```
7. Build V8 shared library

    ```shell
    make s390x -j4 library=shared werror=no
    ```

   The 64-bit shared library will be output as `v8/out/s390x.release/lib.target/libv8.so`.

8. V8 header files will be required to build products invoking the V8 APIs. Issue the following commands to install the header files

    ```shell
    cd /<source_root>/v8
    sudo cp -vR include/* /usr/include/
    ``` 

    Change the file permissions of the following files to 644

    *  /usr/include/libplatform/libplatform.h
    *  /usr/include/v8*h

9. Install the V8 libraries into /usr/local/lib64/ (or /usr/lib64/ if you prefer) for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2

    ```shell
    cd /<source_root>/v8
    sudo cp -v out/s390x.release/lib.target/lib*.so /usr/local/lib64/ 
    sudo chmod -f 755 /usr/local/lib64/libv8.so
    sudo cp -v out/s390x.release/obj.target/src/lib*.a /usr/local/lib64/
    ```
    Follow below Instructions 

    ```shell
    sudo cp -v out/s390x.release/obj.target/third_party/icu/lib*.a /usr/local/lib64/
    sudo chmod -f 755 /usr/local/lib64/libicu*.so
    sudo chmod -f 644 /usr/local/lib64/libv8*.a /usr/local/lib64/libicu*.a
    ```

10. Install the V8 libraries into `/usr/local/lib/` for Ubuntu 16.04/16.10

    ```shell
    cd /<source_root>/v8
    sudo cp -v out/s390x.release/lib.target/lib*.so /usr/local/lib/
    sudo chmod -f 755 /usr/local/lib/libv8.so
    sudo cp -v out/s390x.release/obj.target/src/lib*.a /usr/local/lib/
    ```
    Follow below Instructions 

    ```shell
    sudo cp -v out/s390x.release/obj.target/third_party/icu/lib*.a /usr/local/lib/
    sudo chmod -f 755 /usr/local/lib/libicu*.so
    sudo chmod -f 644 /usr/local/lib/libv8*.a
    ```

11. List shared libraries
    
    ```shell
	sudo /sbin/ldconfig -v
    ```

12. **(Optional)** Check that the sample shell can be invoked and that it displays the expected V8 version. To exit from the shell, enter `quit()`

    ```shell
    /<source_root>/v8/out/s390x.release/v8_shell
    ```

13. **(Optional)** Once you have installed the V8 libraries and header files outside `/<source_root>/` then `/<source_root>/` may be deleted

## References

- [Official website for the V8 JavaScript engine](https://developers.google.com/v8/intro)
- [More information on building the V8 JavaScript engine](https://code.google.com/p/v8/)