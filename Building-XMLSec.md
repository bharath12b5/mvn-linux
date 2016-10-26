<!---PACKAGE:XMLSec--->
<!---DISTRO:SLES 12.x:1.2.x--->
<!---DISTRO:SLES 11.x:1.2.x--->
<!---DISTRO:RHEL 7.x:1.2.x--->
<!---DISTRO:RHEL 6.x:1.2.x--->
<!---DISTRO:Ubuntu 16.x:Distro, 1.2.x--->

# Building XMLSec

Below versions of XMLSec are available in respective distributions:

*    RHEL 6.7     has `1.2.20`
*    RHEL 7.1/7.2     has `1.2.20`
*    Ubuntu 16.04     has `1.2.20`

The instructions provided below specify the steps to build XMLSec 1.2.22 on Linux on the IBM z Systems for RHEL 6.7/7.1/7.2, SLES 11-SP3/12/12-SP1 and Ubuntu 16.04.

_**General Notes:**_ 

i) _When following the steps below please use a standard permission user unless otherwise specified._
	 
ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_

## Building XMLSec

### Obtain pre-built dependencies

1. Use the following commands to obtain dependencies

  For RHEL 7.1/7.2
  
  ```shell
  sudo yum install git make libtool libxslt-devel libtool-ltdl-devel
  ```
  
  For SLES 12/12-SP1
  
  ```shell
  sudo zypper install git-core gcc make libtool libxslt-devel libopenssl-devel
  ```
  
  For RHEL 6.7
  
  ```shell
  sudo yum install tar git make libtool libxslt-devel libtool-ltdl-devel
  ```
  
  For SLES 11-SP3
  
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
