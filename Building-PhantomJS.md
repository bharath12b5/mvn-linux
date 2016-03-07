<!---PACKAGE:PhantomJS--->
<!---DISTRO:SLES 12:2.1.1--->
<!---DISTRO:RHEL 7:2.1.1--->


# Building PhantomJS

PhantomJS can be built for Linux on z System running RHEL 7 or SLES 12 by the following these instructions. Version 2.1.1 has been successfully built & tested this way.

### *General Notes:*
i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `<source_root>` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it.

## Requirements and Prerequisites

### Hardware requirements
* **RAM:** at least 4GB (**Note:** this is critical, building will fail if the memory requirement is not met)
* **Disk space:** at least 3GB
* **CPU:** 1.8 GHz, 4 cores or more

**Tip:** Building PhantomJS from source takes a long time (mainly due to thousands of files in the WebKit module). Estimated build time for a 4-core system is 30 minutes.

### Prerequisites
* **Python:** v2.7.9 (**Optional:** This version is required for running test suite)
    * Instructions for building Python v2.7.9 can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Python-2.7.9)


* **Python Module:** `argparse`
    * To install `argparse` module, run
    ```
    sudo easy_install argparse
    ```

## Building PhantomJS v2.1.1 from source
1.  Install build dependencies

    * For RHEL 7
    ```
    sudo yum -y install gcc gcc-c++ make flex bison gperf ruby \
    openssl-devel freetype-devel fontconfig-devel libicu-devel sqlite-devel \
    libpng-devel libjpeg-devel
    ```
    * For SLES 12
    ```
    sudo zypper install gcc gcc-c++ make flex bison gperf ruby \
    openssl-devel freetype-devel fontconfig-devel libicu-devel sqlite-devel \
    libpng-devel libjpeg-devel
    ```

2.  Retrieve PhantomJS source code
    ```
    git clone git://github.com/ariya/phantomjs.git
    cd phantomjs
    git checkout 2.1.1
    ```

3.  Compile and Link
    ```
    python build.py
    ```
    * **Note:** Building process will take some time. Once it is completed, the executable will be available under the `<source_root>/phantomjs/bin` subdirectory

## Testing (Optional)
1. Run built-in test suite
    ```
    cd <source_root>/test
    python run-tests.py
    ```

2. Test result

    ```
    30.399s elapsed
    203 passed
    4 passed unexpectedly
    12 failed as expected
    2 skipped
    ```
    * **Note**:The built-in test suite was run in a x86 machine as comparison, which shows the same test result.

## Reference
1. [PhantomJS Build Information](http://phantomjs.org/build.html)