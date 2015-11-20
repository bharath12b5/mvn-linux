# Building Apache ZooKeeper

Apache ZooKeeper version 3.5.1 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1/6.6 and SLES 12/11.

### Version
3.5.1

_**General Notes:**_ 	 
_When following the steps below please use a standard permission user unless otherwise specified._

### Section 1: Install the following dependencies
*	git
*	ant
*	ant-junit
*	cppunit-devel
*	cppunit-devel-doc
*   gcc-c++
*   make
*   autoconf
*   libtool


RHEL 7.1/6.6:
```
yum install --nogpgcheck -y \
	git \
	ant \
	hostname \
	ant-junit \
	cppunit-devel \
	cppunit-doc \
	hamcrest-javadoc \
	hamcrest-demo \
	hamcrest

yum groupinstall -y "Development tools"
```

SLES11:
```
zypper install -y git \
	gcc-c++ \
	make \
	tar \
	java-1_7_0-ibm-devel \
	libcppunit-devel \
	libtool

```

SLES12:
```
zypper install -y git \
	ant \
	ant-junit \
	cppunit-devel \
	cppunit-devel-doc \
	gcc-c++ \
	make \
	autoconf \
	libtool 

```

### Install the following additional dependencies for RHEL6.6 and SLES11 (Upgrade the version of automake and autoconf)

RHEL6.6:

        yum install -y wget tar

SLES11:

        zypper install -y wget tar
        wget  wget http://ftp.gnu.org/gnu/m4/m4-1.4.17.tar.xz && tar xf m4-1.4.17.tar.xz
        cd m4-1.4.17 && ./configure &&  make && make install
        
        wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.3-bin.tar.gz
        tar zxf apache-ant-1.9.3-bin.tar.gz    
    
        EXPORT ANT_HOME=/apache-ant-1.9.3
        EXPORT PATH=$PATH:$ANT_HOME/bin
    
SLES11 and RHEL6.6:
   
        wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.xz
        tar xf autoconf-2.69.tar.xz
        cd autoconf-2.69
        ./configure &&  make && make install
        cd /

        wget http://ftp.gnu.org/gnu/automake/automake-1.12.tar.xz
        tar xf automake-1.12.tar.xz
        cd automake-1.12
        ./configure &&  make && make install && cp automake /usr/bin/
        cd /
    




### Section 2: Build and install ZooKeeper
1. Get the source
   
        git clone --branch branch-3.5 https://github.com/apache/zookeeper

2. Install requirements for Build and test case

        cd zookeeper/
   
        ant compile_jute
	
        cd /zookeeper/src/c
	
        ACLOCAL="aclocal -I /usr/share/aclocal" autoreconf -if
    
        ./configure && make && make install
    
        make distclean
	
3. Build and test ZooKeeper

        cd /zookeeper/
        ant jar
        ant test    [optional]
* Note: User can ignore intermittent test-case failures as it does not effect the functionality

## References:
        https://zookeeper.apache.org/