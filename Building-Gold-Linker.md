Gold linker is available on RHEL 7, SLES 12.1 or Ubuntu 16.04. The 2.26 release branch has been built and tested on Linux on z Systems.

1. Install prerequisites:

    For RHEL 7

        sudo yum install bison flex texinfo
        
    For SLES 12
    
        sudo apt-get install bison flex texinfo

    For Ubuntu 16.04
    
        sudo zypper install bison flex texinfo

2. Clone the code and check out the 2.26 branch:

        git clone http://sourceware.org/git/binutils-gdb.git
        cd binutils-gdb
        git checkout binutils-2_26-branch

3. Edit the configure script to allow gold to be built on s390x. See this patch for details:

    https://sourceware.org/git/gitweb.cgi?p=binutils-gdb.git;a=commitdiff;h=ea01647092eefeca9336b36809962ff097306b41

4. Build and install:

        ./configure --prefix=/opt/binutils-2.26 --enable-gold
        make
        sudo make install