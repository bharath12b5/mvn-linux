# Building v8
<!---PACKAGE:V8 JavaScript--->
<!---DISTRO:SLES 12:V8--->
<!---DISTRO:RHEL 7.1:V8--->
<!---DISTRO:Ubuntu 16.x:V8--->

The instructions provided below specify the steps to build V8 JavaScript engine Versions 3.14 and 3.28 on Linux on the IBM z Systems for RHEL 7.1, SLES12 and Ubuntu 16.04. It can be built as a set of shared libraries so that it can be used by multiple applications.
(Note: If you are building V8 for use with [MongoDB](../Building MongoDB), you need V8 3.14.)

More information on the V8 JavaScript engine is available at [the V8 website](https://developers.google.com/v8/intro) and the source code for the z Systems port can be obtained from [GitHub](https://github.com/andrewlow/v8z).

_**General Notes:**_

- When following the steps below, please use a standard permission user unless otherwise specified.

- A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you would like to place it.

## Pre-requisites

1. Issue the following commands to install pre-built dependencies:

    For RHEL 7.1
    ```shell
    sudo yum install git subversion make gcc-c++ tar wget
    ```

    For SLES 12
    ```shell
    sudo zypper install git subversion make gcc-c++ tar wget python-xml
    ```

    For Ubuntu 16.04
    ```shell
	sudo apt-get update
    sudo apt-get install git subversion make tar wget gcc g++ python
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

## Building V8

1. Obtain the V8 source code:

    ```shell
    cd /<source_root>/
    git clone https://github.com/ibmruntimes/v8z
    cd v8z
    ```

2. Select the version to be built:

   (For 3.28)
    
    ```shell
    git checkout 3.28-s390
    ```
     Replace the `builddeps:` section in Makefile
    ```shell

    # Dependencies. "builddeps" are dependencies required solely for building,
    # "dependencies" includes also dependencies required for development.
    # Remember to keep these in sync with the DEPS file.
    builddeps:
     	svn checkout --force http://gyp.googlecode.com/svn/trunk build/gyp \
  	    --revision 1831
  	if svn info third_party/icu 2>&1 | grep -q icu46 ; then \
	  svn switch --force \
	      https://src.chromium.org/chrome/trunk/deps/third_party/icu52 \
	      third_party/icu --revision 277999 ; \
	else \
	  svn checkout --force \
	      https://src.chromium.org/chrome/trunk/deps/third_party/icu52 \
	      third_party/icu --revision 277999 ; \
	fi
	svn checkout --force http://googletest.googlecode.com/svn/trunk \
	    testing/gtest --revision 692
	svn checkout --force http://googlemock.googlecode.com/svn/trunk \
	    testing/gmock --revision 485
    ```
    by

     ```shell
# Dependencies. "builddeps" are dependencies required solely for building,
# "dependencies" includes also dependencies required for development.
# Remember to keep these in sync with the DEPS file.
builddeps:
		git clone https://chromium.googlesource.com/external/gyp build/gyp
	if svn info third_party/icu 2>&1 | grep -q icu46 ; then \
	  svn switch --force \
	      https://src.chromium.org/chrome/trunk/deps/third_party/icu52 \
	      third_party/icu --revision 277999 ; \
	else \
	  svn checkout --force \
	      https://src.chromium.org/chrome/trunk/deps/third_party/icu52 \
	      third_party/icu --revision 277999 ; \
	fi
	svn checkout --force https://github.com/google/googletest.git/trunk/googletest \
	    testing/gtest --revision 951
	svn checkout --force https://github.com/google/googlemock.git/trunk/googlemock \
	    testing/gmock --revision 286
      ```

   (For 3.14)
    
    ```shell
    git checkout 3.14-s390
    ```
    Replace the `builddeps:* in Makefile
     ```shell
    # Dependencies.
    # Remember to keep these in sync with the DEPS file.
    dependencies:
    	svn checkout --force http://gyp.googlecode.com/svn/trunk build/gyp \
    	    --revision 1501
    ```
    by
    ```shell
# Dependencies.
# Remember to keep these in sync with the DEPS file.
dependencies:
		git clone https://chromium.googlesource.com/external/gyp build/gyp
    ```

3. Fetch source code for dependent libraries:

    ```shell
    make dependencies
    ```
4. Build Library
    ```shell

    make s390x -j4
    ```
5. Build the shared library:

    ```shell
    make s390x -j4 library=shared
    ```

   The 64-bit shared library will be output as `v8z/out/s390x.release/lib.target/libv8.so`. When building V8 3.28, libicui18n.so and libicuuc.so are also created in the same directory as libv8.so.

6. V8 header files will be required to build products invoking the V8 APIs. Issue the following commands to install the header files:

    ```shell
    cd /<source_root>/v8z
    sudo cp -vR include/* /usr/include/
    sudo chmod -f 644 /usr/include/v8*h /usr/include/libplatform/libplatform.h
    ```
  
7. Install the V8 libraries into /usr/local/lib64/ (or /usr/lib64/ if you prefer) for RHEL 7/SLES 12 and into `/usr/local/lib/` for Ubuntu 16.04:

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
