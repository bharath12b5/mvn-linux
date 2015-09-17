#Xmlsec 1.2.20_On_RHEL 7.1/6.6_SLES 12/11
Xmlsec code can be built for Linux on z Systems running RHEL 7.1/6.6 or SLES 12/11 by following these instructions.  Version 1.2.20 has been successfully built & tested this way.

_**General Notes:**_ 

i) _When following the steps below please use a standard permission user unless otherwise specified._
	 
ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_

## Building Xmlsec

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

### Product Build - Xmlsec

1. Create a working directory with write permission to use as an installation workspace (Referred to as `/<source_root>/` from this point on) :

  ```shell
	mkdir /source_root/
	cd /source_root/
  ```

2. Download Xmlsec source code

  ```shell
  cd /<source_root>/
  git clone https://github.com/GNOME/xmlsec.git
  cd xmlsec
  ```
  
3. Generate and then run the configuration

  ```shell
  ./autogen.sh
  ```
  
4. Build and (_optionally_) test

  ```shell
  make
  ```

5. Install xmlsec and verify the installation

  ```shell
  sudo make install
  make check
  ```
  
  _**Note:**_ If the following error is shown, set the environment variable "LD_LIBRARY_PATH" to path of xmlsec binaries e.g. `export LD_LIBRARY_PATH=/usr/local/lib`

  ```shell
  xmlsec1: error while loading shared libraries: libxmlsec1.so.1: cannot open shared object file: No such file or directory
  ```