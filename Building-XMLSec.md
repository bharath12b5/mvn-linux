# Building XMLSec
These steps are tested on sles12 and rhel7 on IBM z systems.

###Version
>1.2.20
####1.  Install the following dependencies:
*	git-core or git
*	libtool
*	make
*	gcc
*	libxslt-devel
*	libtool-ltdl-devel (on rhel)

RHEL7:
```
yum install -y \
	  git \
	  make \
	  libtool \
	  libxslt-devel \
	  libtool-ltdl-devel
```		

SLES12

```
zypper install -y \
	  git-core \
	  gcc \
	  make \
	  libtool \
	  libxslt-devel \
	  libopenssl-devel-1.0.1i-2.12.s390x
```
####2.  Clone source repository:
    git clone https://github.com/GNOME/xmlsec.git

####3.  Build and install:
         cd xmlsec
        ./autogen.sh
        make
        make install

####4.  Run test cases:
     make check

>Note:If the following error is shown
xmlsec1: error while loading shared libraries: libxmlsec1.so.1: cannot open shared object file: No such file or directory
Then set the environment variable "LD_LIBRARY_PATH" to path of xmlsec binaries
EG:export LD_LIBRARY_PATH=/usr/local/lib
