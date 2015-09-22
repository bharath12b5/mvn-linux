This recipe describes how to build Jblas on Linux on z Systems.

1. As root, install Fortran:

        sudo zypper install gcc-fortran libgfortran3
  

2. Download and unpack the latest stable `lapack` source code:

        wget http://www.netlib.org/lapack/lapack-lite-3.1.1.tgz
        tar zxvf lapack-lite-3.1.1.tgz
        cd lapack-lite-3.1.1


3. The makefile in lapack includes a file named `make.inc` which defines variables for the build platform. Create this file by pasting the following content into `make.inc`:

    ```shell
    FORTRAN  = gfortran
    OPTS     = -funroll-all-loops -O3
    DRVOPTS  = $(OPTS)
    NOOPT    =
    LOADER   = gfortran
    LOADOPTS =
    TIMER    = INT_CPU_TIME
    ARCH     = ar
    ARCHFLAGS= cr
    RANLIB   = ranlib
    BLASLIB      = ../../libblas$(PLAT).a
    LAPACKLIB    = liblapack$(PLAT).a
    TMGLIB       = libtmglib$(PLAT).a
    EIGSRCLIB    = libeigsrc$(PLAT).a
    LINSRCLIB    = liblinsrc$(PLAT).a
    ```

4. Compile and build the libraries:

    ```shell
    make blaslib
    make all
    export LAPACK_HOME=`pwd`
    ```

5. Download the latest stable jblas source code:

    ```shell
    wget https://github.com/mikiobraun/jblas/archive/jblas-1.2.4.tar.gz
    tar zxvf jblas-1.2.4.tar.gz
    cd jblas-jblas-1.2.4/
    ```

6. Configure the program and `make` it:

    ```shell
    ./configure --static-libs --built-type=lapack --lapack-build --libpath=$LAPACK_HOME
    make
    mvn package install
    ls -la target/jblas-1.2.4*
    ```
7. Put the .so files into the .jar

   This will place the jblas JAR file in the local Maven repository ($HOME/.m2)