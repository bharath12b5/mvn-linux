#Building ProtoBuf 2.6.1

This recipe covers Protobuf v2.6.1 built on SLES 12 and RHEL7.1 on IBM z Systems.

The following build instructions have been tested with **protobuf2.6.1** on SLES 12 and RHEL7.1 on IBM z Systems.

####Step 1:Install the Dependencies
Following are the build dependencies for installing Protobuf.
*		git (RHEL7) / git-core (SLES12)
*       autoconf
*       libtool
*       make
*       gcc-c++
*       curl
*       automake
*   	tar
*   	bzip2

RHEL7:
```
yum -y update && yum install -y \
    git \
    tar \
    autoconf \
    libtool \
    make \
    gcc-c++ \
    bzip2
```

SLES12:
```
zypper install -y \
	autoconf \
	libtool \
	automake \
	gcc-c++ \
	make \
	curl \
	git-core
```

####Step 2: Clone the repository and checkout v2.6.1
Download  source from git: [Protobuf]
```
	git clone https://github.com/google/protobuf.git
    cd protobuf
    git checkout v2.6.1
```

####Step 3: Building and Installing Protobuf 2.6.1
1. Generate the configuration script
```
./autogen.sh
```
2. Configure the environment
```
./configure
```
3. Compile all the files in the code base
```sh
make
```
4. Run the test suite
```sh
make check
```
5. Install protobuf
```sh
make install
```

###Verification:
To verify run `protoc --help`. It should display the options and Usage of protoc.

#####Note:
By default, the package will be installed to /usr/local. However, on many platforms, /usr/local/lib is not part of LD_LIBRARY_PATH. You can add it, but it may be easier to just install to /usr instead. To do this, invoke configure as follows:


		./configure --prefix=/usr
            (or)
		export LD_LIBRARY_PATH=/usr/local/lib

##References:
https://github.com/google/protobuf

[Protobuf]:https://github.com/google/protobuf.git
