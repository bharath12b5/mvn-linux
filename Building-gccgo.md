These instructions describe how to build gccgo 1.4.2 for IBM Linux on z Systems (RHEL 6.6/RHEL 7.1 and SLES 11/SLES12).

[GCC 5](https://gcc.gnu.org/gcc-5/changes.html) provides a complete implementation of Go 1.4.2. Until GCC 5 is officially released for RHEL and SLES, you can build the GCC source code to obtain a Go 1.4.2 compiler and run-time environment for testing on Linux on z Systems.  These instructions have been tested on both RHEL 6.6/7.1 and SLES 11/12.

_**Note:** When following the steps below, please use a standard permission user unless otherwise specified._

## Building and Installing GCC

1. Install the prerequisites for building GCC.

    RHEL
   ```shell
   sudo yum install wget tar make flex subversion gcc gcc-c++ binutils-devel bzip2
   ```
    SLES
    ```shell
   sudo zypper install wget tar make flex subversion gcc gcc-c++ binutils-devel bzip2
   ```

 _**Note:** gcc source includes`download_prerequisites`, a script to install MPFR, MPC, GMP, and ISL for the gcc version being built._

2. Create a working directory with write permission to use as a `gccgo` installation workspace (Referred to as `/<source_root>/` from this point on) :

   ```shell
   mkdir /<source_root>/
   cd /<source_root>/
   ```

 _**Note :** Creation of a separate working directory is recommended to allow safe removal of the transient working data, which minimises risk to the rest of the file system._

3. Check-out a GCC source version from the official subversion repository into a sub-directory `gcc_srcDir` :

 The latest gcc source is subject to continual updates, leading to occasional build problems that are beyond the scope of this recipe. This recipe was tested, and shown to work at revision `223813`.  Please choose only one of the `gcc` download options offered here :-

 **EITHER** Download the previously tested 223813 svn revision  :
   ```shell
   svn co svn://gcc.gnu.org/svn/gcc/trunk@223813 gcc_srcDir
   ```

 **OR ALTERNATIVELY** Download the latest gcc source version, (or an alternative gcc version of your own choosing)  but if problems are experienced, the suggested first test would be to repeat the recipe using the 223813 revision.
   ```shell
   svn co svn://gcc.gnu.org/svn/gcc/trunk gcc_srcDir
   ```

4. The gcc `download_prerequisites` script should be located within a `contrib` directory. Make sure it is run it from the top source tree level, to ensure the pre-requisites are found at build time :

    ```shell
    cd /<source_root>/gcc_srcDir
    ./contrib/download_prerequisites
    ```
_**Note :** GCC tends to have problems when configured in the same directory as the GCC source code, or in any subdirectory therein, so the build process is done from a peer_level `gcc_buildDir/build` directory._

5. Create a `gcc_buildDir/build` directory to configure and make software to include `c` and `go` :

    ```shell
    mkdir -p /<source_root>/gcc_buildDir/build
    cd /<source_root>/gcc_buildDir/build

    ../../gcc_srcDir/configure \
    --prefix="/opt/gccgo" \
    --enable-shared --with-system-zlib --enable-threads=posix \
    --disable-multilib --enable-__cxa_atexit --enable-checking \
    --enable-gnu-indirect-function --enable-languages="c,c++,go" \
    --disable-bootstrap

    make all
    ```

 _**Note:** By changing the prefix, you can specify a different installation directory._

6. As root user, install the binaries (into /opt/gccgo) :

    ```shell
    sudo make install
    ```

7. Change directory away from `<source_root>` so it can be deleted :
    ```shell
    cd /<home>/
    sudo rm -rf /<source_root>/
    ```
_**Note :** The reason for deleting `/<source_root>/` is to ensure it cannot contaminate the testing. An alternative approach might be to `zip` the data into an archive file, allowing later re-use or examination._

8. To use the `go` utility, the user PATH and shared library cache needs to be updated :

    ```shell
    export PATH=/opt/gccgo/bin:$PATH

    sudo sh -c "echo '/opt/gccgo/lib64' >> /etc/ld.so.conf.d/gccgo.conf"
    sudo /sbin/ldconfig
    ```
  _**Note:** sudo has no write access on `/etc/ld.so.conf.d` so the `gccgo.conf` file was created via a sudo shell.  The raw use of `sudo ldconfig` creates necessary links and cache to shared libraries. Comparing outputs from the commands `/sbin/ldconfig -p | grep gccgo` and `/sbin/ldconfig -v | grep gccgo` before and after the library settings  will confirm the changes have taken effect._

## Compiling and running Go programs with gccgo

9. **OPTIONAL** A basic 'Hello World' program to demonstrate the `go` compiler :

    Create a text file  'HelloWorld.go' containing the following:

    ```code
    package main

    import "fmt"

    func main() {
        fmt.Println("Hello World")
    }
    ```

10. First run the 'Hello World' program, then build an executable from it, which can also be run :

   ```shell
   go run ./HelloWorld.go

   go build --compiler=gccgo -o myTestProgram HelloWorld.go
   ./myTestProgram
   ```

11. Run `ldd` on `myTestProgram` to show the dynamic dependencies are satisfied :

    ```shell
    ldd ./myTestProgram
    ```
_**Note:** The command output should be similar to the sample output below. Some digits and version numbers may vary slightly, but it should be apparent if some of the dependencies are not satisfied, indicating a problem._

  ```shell
  libgo.so.7 => /opt/gccgo/lib64/libgo.so.7 (0x000003fffc1aa000)
  libm.so.6 => /lib64/libm.so.6 (0x000003fffc103000)
  libgcc_s.so.1 => /opt/gccgo/lib64/libgcc_s.so.1 (0x000003fffc0f1000)
  libc.so.6 => /lib64/libc.so.6 (0x000003fffbf37000)
  libpthread.so.0 => /lib64/libpthread.so.0 (0x000003fffbf14000)
  /lib/ld64.so.1 (0x000002aae9e4b000)
  ```

11. To build Go binaries that do not require the Go run-time library (so that they can run on a different system without gccgo installed), give gccgo the `-static-libgo` flag, e.g.

  ```shell
  go build -compiler=gccgo -gccgoflags='-static-libgo' -o myTestProgram HelloWorld.go
  ```

12. And finally, verify the go and gcc version with the following command:

  ```shell
  go version
  ```
  _**Note:** It is only important that the output corresponds with the version you have built. The minor version 20150528 refers to a svn build, and the (GCC) 6.0.0 refers to the gcc version gccgo was built with._

  ```shell
  go version go1.4.2 gccgo (GCC) 6.0.0 20150528 (experimental) linux/s390x
  ```
