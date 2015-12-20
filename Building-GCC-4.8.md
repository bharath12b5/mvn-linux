[GCC](https://gcc.gnu.org/) 4.8.3 is shipped with newer versions of enterprise Linux distributions (RHEL 7 and SLES 12). However, older versions of the distributions, RHEL 6 and SLES 11, only provide GCC 4.4.7 and GCC 4.3.4 by default, respectively. While the SLES 11 SP4 SDK provides optional packages that enable users to use GCC 4.8 (e.g. `zypper install gcc48-c++`), RHEL 6 provides no such option. To be able to build software with GCC 4.8 on RHEL 6 on z Systems, you must build and install GCC yourself. This recipe will cover how to build and install GCC 4.8.5 on RHEL 6.

1. Install prerequisite packages for building GCC:

        sudo yum install wget tar make flex gcc gcc-c++ gcc-devel.s390 binutils-devel bzip2

1. Create a new working directory to use as a build and installation workspace. This is recommended to allow safe removal of the transient working data, which minimizes risk to the rest of the file system:

        mkdir /<source_root>/
        cd /<source_root>/

1. Download the GCC source tarball from the official FTP server into the `/<source_root>/` directory:

        wget ftp://gcc.gnu.org/pub/gcc/releases/gcc-4.8.5/gcc-4.8.5.tar.gz

1. Extract the tarball; this will create the directory `/<source_root>/gcc-4.8.5/`:

        tar xzf gcc-4.8.5.tar.gz

1. Run the download_prerequisites script to install various required libraries (MPFR, MPC, GMP, and ISL) for the GCC version being built. Make sure it is run from the top-level directory, to ensure that the prerequisites are placed in the correct locations and can be found at build time:

        cd /<source_root>/gcc-4.8.5/
        ./contrib/download_prerequisites

1. Configure and build GCC. **Note**: GCC tends to have problems when configured in the same directory as the GCC source code, so the build process is done in a separate build directory:

        mkdir -p /<source_root>/gccbuild
        cd /<source_root>/gccbuild

        ../gcc-4.8.5/configure \
        --prefix="/opt/gcc" \
        --enable-shared --with-system-zlib --enable-threads=posix \
        --enable-__cxa_atexit --enable-checking --enable-gnu-indirect-function \
        --enable-languages="c,c++" --disable-bootstrap

        make all

   You can specify an installation directory other than /opt/gcc, by changing the path name passed to the `--prefix` option.

1. As root user, install the binaries:

        sudo make install

1. **(Optional)** Clean up the build workspace:

        cd /
        sudo rm -rf /<source_root>/

1. To use the new version of GCC, the user's `PATH` environment variable needs to be updated to include the new directory:

        export PATH=/opt/gcc/bin:$PATH

   Running `gcc -v` should show that GCC 4.8.5 is being used.

1. Programs built with the new version of GCC may depend on runtime features that only exist in that version. To run such programs correctly, instruct the dynamic linker to look in the correct location by setting the `LD_LIBRARY_PATH` environment variable:

        export LD_LIBRARY_PATH='/opt/gcc/$LIB'

   **Note:** The `$LIB` pattern must be specified as-is in the value of the `LD_LIBRARY_PATH` variable (hence the single quotes). It will be expanded by the dynamic linker to either "lib" or "lib64", depending on the addressing mode of the program being executed.