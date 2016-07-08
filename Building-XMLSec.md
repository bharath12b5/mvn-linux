<!---PACKAGE:XMLSec--->
<!---DISTRO:SLES 12:1.2.22--->
<!---DISTRO:SLES 11:1.2.22--->
<!---DISTRO:RHEL 7.1:1.2.22--->
<!---DISTRO:RHEL 6.6:1.2.22--->
<!---DISTRO:Ubuntu 16.x:1.2.22--->

# Building XMLSec

Below versions of XMLSec are available in respective distributions:

*    RHEL 6     has `1.2.20`
*    RHEL 7     has `1.2.20`
*    Ubuntu 16.04     has `1.2.20`

The instructions provided below specify the steps to build XMLSec 1.2.22 on Linux on the IBM z Systems for RHEL 6/7, SLES 11/12 and Ubuntu 16.04.

_**General Notes:**_ 

i) _When following the steps below please use a standard permission user unless otherwise specified._
	 
ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_

## Building XMLSec

### Obtain pre-built dependencies

1. Use the following commands to obtain dependencies

  For RHEL 7.1
  
  ```shell
  sudo yum install git make libtool libxslt-devel libtool-ltdl-devel
  ```
  
  For SLES 12
  
  ```shell
  sudo zypper install git-core gcc make libtool libxslt-devel libopenssl-devel-1.0.1i-2.12.s390x
  ```
  
  For RHEL 6.6
  
  ```shell
  sudo yum install tar git make libtool libxslt-devel libtool-ltdl-devel
  ```
  
  For SLES 11
  
  ```shell
  sudo zypper install tar pkg-config autoconf automake git-core gcc make libtool libxslt-devel
  ```
  
  For Ubuntu 16.04
  
  ```shell
  sudo apt-get update
  sudo apt-get install git make libtool libtool-bin libxslt1-dev autoconf libxmlsec1-dev
  ```

### Product Build - XMLSec

1. Create a working directory with write permission to use as an installation workspace (Referred to as `/<source_root>/` from this point on) :

  ```shell
	mkdir /source_root/
	cd /source_root/
  ```

2. Download XMLSec source code

  ```shell
  cd /<source_root>/
  git clone https://github.com/lsh123/xmlsec.git
  cd xmlsec
  git checkout xmlsec-1_2_22
  ```
  
3. Generate and then run the configuration

  ```shell
  ./autogen.sh
  ```
  
4. Build and (_optionally_) test

  ```shell
  make
  ```

5. Install XMLSec and verify the installation

  ```shell
  sudo make install
  make check
  ```
  
  _**Note:**_ If the following error is shown, set the environment variable "LD_LIBRARY_PATH" to path of XMLSec binaries e.g. `export LD_LIBRARY_PATH=/usr/local/lib`

  ```shell
  xmlsec1: error while loading shared libraries: libxmlsec1.so.1: cannot open shared object file: No such file or directory
  ```
  or
  ```shell
  xmlsec1: symbol lookup error: xmlsec1: undefined symbol: xmlSecGetDefaultCrypto
```
  
# References  
  https://github.com/GNOME/xmlsec
