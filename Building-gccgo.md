[GCC 5](https://gcc.gnu.org/gcc-5/changes.html) provides a complete implementation of Go 1.4.2. Until GCC 5 packages are officially available from distributions, you can build the GCC source code to obtain a Go 1.4.2 compiler and run-time environment to use on Linux on z Systems. These instructions have been tested on RHEL 6, RHEL 7.1, SLES 11 SP3 and SLES 12.

## Building and installing GCC 5

1. Install the [prerequisites](https://gcc.gnu.org/install/prerequisites.html) for building GCC:

   (SLES 12)

        sudo zypper install gmp-devel binutils-devel mpfr-devel mpc-devel isl-devel
        sudo zypper install flex subversion

   (RHEL 7)

        sudo yum install gmp-devel binutils-devel mpfr-devel libmpc-devel
        sudo yum install flex subversion
        wget ftp://gcc.gnu.org/pub/gcc/infrastructure/isl-0.14.tar.bz2

   RHEL 7.1 does not provide the ISL library as an RPM  package, so it needs to be downloaded in source form and built into GCC.

   Subversion is only required if you plan to build the latest development source code.

   If you are running an older version of Linux (e.g. RHEL 6 or SLES 11), the gmp, isl, mpc, and mpfr dependencies can be downloaded into the GCC source tree with a script; see step 3 below.

2. Download the [GCC 5.1.0 source code](ftp://gcc.gnu.org/pub/gcc/releases/gcc-5.1.0/gcc-5.1.0.tar.bz2) and unpack it:

        tar xjf gcc-5.1.0.tar.bz2
        cd gcc-5.1.0

   Alternatively, if you wish to build the latest development source code, check out the GCC source code from the official code repository, into a new directory named "gcc":

        svn co svn://gcc.gnu.org/svn/gcc/trunk gcc
        cd gcc

3. On RHEL 7.1 only, unpack the ISL source files within the GCC source tree, and rename (or symlink) the directory to `isl`:

        tar xjvf ../isl-0.14.tar.bz2
        ln -s isl-0.14 isl

   If you are building on RHEL 6 or SLES 11, run the following script to obtain compatible versions of the dependencies:

        ./contrib/download_prerequisites

   This will create the gmp, isl, mpc and mpfr directories within the GCC source trees.

4. Now configure and build the C and Go compilers:

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

        sudo make install

## Compiling and running Go programs with gccgo

1. Make sure the gccgo binaries are on your $PATH:

        export PATH=/opt/gccgo/bin:$PATH
        export LD_LIBRARY_PATH=/opt/gccgo/lib64

2. Build Go programs as usual, but add the `-compiler=gccgo` option to the `go` command line:

        go build -compiler=gccgo -o helloworld helloworld.go

3. The resulting binaries are dynamically linked to the Go run-time library (libgo.so), e.g.

        $ ldd helloworld
        libgo.so.7 => /opt/gccgo/lib64/libgo.so.7 (0x000003fffc114000)
        libm.so.6 => /lib64/libm.so.6 (0x000003fffd5dc000)
        libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x000003fffd5ca000)
        libc.so.6 => /lib64/libc.so.6 (0x000003fffd429000)
        /lib/ld64.so.1 (0x000002aac322e000)

   To make it easier to run Go programs, we recommend adding /opt/gccgo/lib64/ to the dynamic linker's configuration (as root). This can be achieved by editing /etc/ld.so.conf or by adding a new file under /etc/ld.so.conf.d/, and then configuring the dynamic linker, e.g.

        echo /opt/gccgo/lib64 > /etc/ld.so.conf.d/gccgo.conf
        ldconfig -v

4. To build Go binaries that do not require the Go run-time library, give gccgo the `-static-libgo` flag, e.g.

        go build -compiler=gccgo -gccgoflags='-static-libgo' -o helloworld helloworld.go