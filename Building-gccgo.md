[GCC 5](https://gcc.gnu.org/gcc-5/changes.html) will provide a complete implementation of Go 1.4.2. Until GCC 5 is officially released later this year, you can build the GCC source code to obtain a Go 1.4.2 compiler and run-time environment for testing on Linux on z Systems. These instructions have been tested on both RHEL 7 and SLES 12.

### Building and installing GCC 5

1. Install the [prerequisites](https://gcc.gnu.org/install/prerequisites.html) for building GCC:

   (SLES 12)

        sudo zypper install gmp-devel binutils-devel mpfr-devel mpc-devel isl-devel
        sudo zypper install flex subversion

   (RHEL 7)

        yum install gmp-devel binutils-devel mpfr-devel libmpc-devel
        yum install flex subversion
        wget ftp://gcc.gnu.org/pub/gcc/infrastructure/isl-0.14.tar.bz2

   RHEL 7 does not provide the ISL library as a installable package, so it needs to be downloaded in source form and built into GCC.
  
2. Check out the latest GCC 5 source code from the official code repository, into a new directory named "gcc":

        svn co svn://gcc.gnu.org/svn/gcc/trunk gcc

3. On RHEL 7 only, unpack the ISL source files and move them into the GCC source tree:

        tar xjvf isl-0.14.tar.bz2
        mv isl-0.14 gcc/isl

4. Now configure and build the C and Go compilers:

        cd gcc
        mkdir build
        cd build
        ../configure \
            --prefix="/opt/gccgo" \
            --enable-shared --with-system-zlib --enable-threads=posix \
            --disable-multilib --enable-__cxa_atexit --enable-checking \
            --enable-gnu-indirect-function --enable-languages="c,go" \
            --disable-bootstrap
        make all

   You may want to specify a different installation directory instead of /opt/gccgo/.

5. Install the GCC binaries into /opt/gccgo/:

        make install

### Compiling and running Go programs with gccgo

1. Make sure the GCC binaries are on your $PATH:

        export PATH=/opt/gccgo/bin:$PATH
        export LD_LIBRARY_PATH=/opt/gccgo/lib64

2. Build Go programs as usual, but add the `--compiler=gccgo` option to the `go` command line:

        go build --compiler=gccgo -o helloworld helloworld.go

3. The resulting binaries are dynamically linked to the Go run-time library, e.g.

        $ ldd helloworld
        libgo.so.7 => /opt/gccgo/lib64/libgo.so.7 (0x000003fffc114000)
        libm.so.6 => /lib64/libm.so.6 (0x000003fffd5dc000)
        libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x000003fffd5ca000)
        libc.so.6 => /lib64/libc.so.6 (0x000003fffd429000)
        /lib/ld64.so.1 (0x000002aac322e000)

   To make it easier to run Go programs, we recommend adding /opt/gccgo/lib64/ to the dynamic linker's configuration (as root). This can be achieved by editing /etc/ld.so.conf or by adding a new file under /etc/ld.so.conf.d/, and then configuring the dynamic linker, e.g.
  
        echo /opt/gccgo/lib64 > /etc/ld.so.conf.d/gccgo.conf
        ldconfig -v