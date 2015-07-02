# Building Apache Http Web Server

The following build instructions have been tested with Apache HTTP 2.5 on RHEL 7.1/6.6 and SLES 12/11 on IBM Linux on z Systems.

_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified._  
_**NOTE:** A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_  
_**NOTE:** For convenience `vi` has been used in the instructions below when editing files, replace with your desired editing program if required_

1. Install build dependencies

    For RHEL 7.1 & 6.6
    ```shell
    yum install git openssl openssl-devel gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel expat-devel which wget tar
    ```
    And for SLES 12 & 11
    ```shell
    zypper install git openssl openssl-devel gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel wget tar
    ```
2. SLES 11 & RHEL 6.6 **Only** - build libtool

    SLES 11 and RHEL 6.6 have too old a version of libtool available for Apache HTTP Server to build successfully so you need to build a later version
    
    1. Download libtool source code
    
        ```shell
        cd /<source_root>/
        wget http://ftpmirror.gnu.org/libtool/libtool-2.4.6.tar.gz
        tar -xvf libtool-2.4.6.tar.gz
        cd libtool-2.4.6
        ```
    2. Configure and make
    
        ```shell
        ./configure
        make
        ```
    4. Install libtool
    
        ```shell
        sudo make install
        ```
3. Extract Apache HTTP source code (and supporting packages)

    ```shell
    cd /<source_root>/
    git clone https://github.com/apache/httpd.git
    cd httpd/srclib
    git clone https://github.com/apache/apr.git
    ```
4. Build Apache Portable Runtime

    ```shell
    cd apr
    ./buildconf 
    ./configure
    make
    sudo make install
    ```
5. Build HTTPD

    ```shell
    cd /<source_root>/httpd
    ./buildconf
    ./configure --prefix=<build-location>
    make
    ```
    _**Note:** Skipping the `--prefix` results in Apache httpd being installed in the default location_
7. Install

    ```shell
    sudo make install
    ```
8. Verification

    _**Note:** All the following commands may require `sudo` depending on the `<build-location>` specified_  
    
    Update your configuration as necessary
    ```shell
    vi <build-location>/conf/httpd.conf
    ```
    Verify the configuration and start the server
    ```shell
    <build-location>/bin/apachectl configtest
    <build-location>/bin/apachectl -k start
    ```
    _Test the webserver_
    ```shell
    <build-location>/bin/apachectl -k stop
    ```
    _**Note:** `<build-location>` is the prefix you specified on step 5 - if you didn't specify a prefix it should be installed to `/usr/local/apache2`_  
    

##References:
https://github.com/apache/httpd	
