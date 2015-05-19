#Building Xerces-C 3.1

Recipe for xerces-c version 3.1.

The following build instructions have been tested with **xerces-c 3.1** on SLES12 and RHEL7 on IBM z Systems.

####Step 1:Install the Dependencies
Following are the build dependencies for installing xerces-c. 
*		git (RHEL7) / git-core (SLES12)
*       libtool
*       make
*       gcc-c++
*       automake

RHEL7:
```
yum -y update && yum install -y \
    git \
    automake \
    libtool \
    make \
    gcc-c++
```

SLES12:
```
zypper install -y \
   libtool \
   automake \
   gcc-c++ \
   make \
   git-core
```

####Step 2: Clone the repository and checkout version 3.1
Download  source from git: [Xerces-C]
```
    git clone https://github.com/apache/xerces-c.git --branch xerces-3.1
```

####Step 3: Set Environment variables

By default System LANGUAGE is set to C, unless you explicitly set it. You can verify it by running ```locale```. Export system language settings.

SLES12:

```
    export LANG='en_US.UTF-8'
    export LANGUAGE='en_US.UTF-8'
```

RHEL7:

You may face issues while setting system language. To avoid that run
```yum reinstall -y glibc-common```

```
    export LANG='en_US.UTF-8'
    export LC_ALL='en_US.UTF-8'
```

If language is not set properly tests and installation fails due to encoding issues. Please install **locale** if its not previously installed.

####Step 4: Build and Install Xerces-C
1. Generate the configuration script
```
./reconf
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
5. Install Xerces-C
```sh
make install
```

###Verification:
As Xerces_c is a XML parser it doesnt have a specific commandline. It has libraries which you can find in `LD_LIBRARY_PATH` of your machine.

##References:
https://xerces.apache.org/xerces-c/build-3.html#UNIX

[Xerces-C]:https://github.com/apache/xerces-c.git
