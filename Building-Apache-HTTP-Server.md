<!---PACKAGE:Apache HTTP--->
<!---DISTRO:SLES 12:2.4--->
<!---DISTRO:SLES 11:2.4--->
<!---DISTRO:RHEL 7.1:2.4--->
<!---DISTRO:RHEL 6.6:2.4--->
<!---DISTRO:Ubuntu 16.x:2.4--->

#Building Apache HTTP Server

Below versions of Apache HTTP Server are available in respective distributions at the time of this recipe creation:
*    RHEL 6.8 has `2.2.15`
*    RHEL 7.1/7.2/7.3 have `2.4.6`
*    SLES 11-SP4 has `2.2.12`
*    SLES 12 has `2.4.10`
*    SLES 12-SP1 and 12-SP2 has `2.4.16`
*    Ubuntu 16.04 has `2.4.18`

The instructions provided below specify the steps to build Apache HTTP Server version 2.4.25 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.  

_**General Notes:**_
* When following the steps below please use a standard permission user unless otherwise specified.
* A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

##Step 1: Building Apache HTTP Server
####1.1) Install dependencies
* RHEL 6.8 and RHEL 7.1/7.2/7.3
  ```sh
  sudo yum install git openssl openssl-devel gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel expat-devel which wget tar
  ```

* SLES 11-SP4
  ```sh
  sudo zypper install git gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel wget tar awk
  ```

* SLES 12/12-SP1/12-SP2
  ```sh
  sudo zypper install git openssl openssl-devel gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel wget tar
  ```

* Ubuntu 16.04/16.10
  ```sh
  sudo apt-get update
  sudo apt-get install git python openssl gcc autoconf make libtool-bin libpcre3-dev libxml2  libexpat1 libexpat1-dev wget tar 
  ```
  
####1.2) Build Openssl 1.0.2 (SLES 11-SP4 Only)
```sh
cd /<source_root>/
wget ftp https://www.openssl.org/source/old/1.0.2/openssl-1.0.2i.tar.gz
tar zxf openssl-1.0.2i.tar.gz
cd openssl-1.0.2i
./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib shared zlib-dynamic
make
sudo make install		
export PATH=/usr/local/ssl/bin:$PATH
```

####1.3) Build Libtool (RHEL 6.8 andSLES 11-SP4 Only)
```sh
cd /<source_root>/
wget http://ftpmirror.gnu.org/libtool/libtool-2.4.6.tar.gz
tar -xvf libtool-2.4.6.tar.gz
cd libtool-2.4.6
./configure
make
sudo make install
```

_**Note:** RHEL 6.8 and SLES 11-SP4 has an older version of libtool available. Apache HTTP Server needs a later version of libtool._

####1.4) Extract Apache HTTP source code (and supporting packages)
```sh
cd /<source_root>/
git clone https://github.com/apache/httpd.git 
cd /<source_root>/httpd
git checkout 2.4.25
cd /<source_root>/srclib
git clone https://github.com/apache/apr.git
```

####1.5) Build Apache Portable Runtime
```sh
cd /<source_root>/httpd/apr
./buildconf 
./configure
make
sudo make install
```

_**Note:** The apache2-utils binaries are created in `/<source_root>/httpd/support/` folder. Following is the list of binaries that are created:_
* ab 
* logresolve
* htpasswd
* htdigest
* htdbm
* rotatelogs
* checkgid
* dbmmanage
* split-logfile
* check_forensic
* httxt2dbm

####1.6) Build and Install Apache HTTP Server
```sh
cd /<source_root>/httpd
./buildconf
./configure --prefix=<build-location>
make
sudo make install
```

_**Note:** Skipping the `--prefix` results in Apache httpd being installed in the default location._

##Step 2: Verification(Optional)

_**Note:** All the following commands may require `sudo` depending on the `<build-location>` specified._

####2.1) Update your configuration as necessary

To update configuration you may modify `<build-location>/conf/httpd.conf` file.

####2.2) Verify the configuration and start the server
```sh
<build-location>/bin/apachectl configtest
<build-location>/bin/apachectl -k start
```

_**Note:** If `<build-location>` is the prefix you specified, ensure that in <build-location>/conf/httpd.conf file, values for `User` and `Group` fields are same as that of `<build-location>`. Else, it will display forbidden(403) error in the browser._  
    
####2.3) Stop the webserver
```sh
<build-location>/bin/apachectl -k stop
```

_**Note:** `<build-location>` is the prefix you specified, if you didn't specify a prefix it should be installed to `/usr/local/apache2`._

##References:
http://httpd.apache.org/
