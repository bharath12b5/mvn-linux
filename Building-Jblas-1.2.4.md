**jblas** is a fast linear algebra library for Java. jblas is based on BLAS and LAPACK, the de-facto industry standard for matrix computations, and uses state-of-the-art implementations like ATLAS for all its computational routines, making jBLAS very fast.

This recipe describes how to build Jblas on Linux on z Systems.

1. As root, install Fortran:

        sudo zypper install gcc-fortran libgfortran3
  
2. Download the latest stable jblas source code:

    ```shell
    wget https://github.com/mikiobraun/jblas/archive/jblas-1.2.4.tar.gz
    tar zxvf jblas-1.2.4.tar.gz
    cd jblas-jblas-1.2.4/
    export JBLAS_HOME=`pwd`
    ```

3. Download the dependency of LAPACK through jblas's configure program:

    ```shell
    ./configure --download-lapack
    tar zxvf lapack-lite-3.1.1.tgz
    cd lapack-lite-3.1.1/
    export LAPACK_LITE=`pwd`
    ```

4. The makefile in lapack includes a file named `make.inc` which defines variables for the build platform. Create this file by pasting the following content into `make.inc`:

    ```shell
    FORTRAN  = gfortran
    OPTS     = -fPIC -funroll-all-loops -O3
    DRVOPTS  = $(OPTS)
    NOOPT    = -fPIC
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

5. Compile and build the libraries:

    ```shell
    make blaslib
    make all
    ```

6. Change directory back to jblas, then run configure:

    ```shell
    cd $JBLAS_HOME
    ./configure --static-libs --built-type=lapack --lapack-build --libpath=$LAPACK_HOME
    ```

7. Modify the configure output file, `configure.out`, look for the variable `LOADLIBES` and make sure it looks like: 

        LOADLIBES=-Wl,-z,muldefs ./lapack-lite-3.1.1/liblapack.a ./lapack-lite-3.1.1/libblas.a -lgfortran

8. Run make to build and install Jblas:

    ```shell
    make
    mvn package install
    ls -la target/jblas-1.2.4*
    ```