[gperftools](https://github.com/gperftools/gperftools) is a collection of performance tools for Linux, including a thread-caching memory allocator (tcmalloc), a CPU profiler, a heap profiler, and a heap checker. These tools can be linked into a program, in order to improve its memory allocation performance, and observe its run-time behaviour by profiling.

The latest development version of gperftools has been ported to Linux on z Systems. The port is being contributed upstream; until it has been released officially, you can build and install gperftools on RHEL 7 and SLES 12 on z Systems using the following build instructions.

1. Install prerequisites:

   (RHEL)

        sudo yum install git autoconf automake libtool

   (SLES)

        sudo zypper install git autoconf automake libtool
      
1. Clone the gperftools source code:

        git clone https://github.com/linux-on-ibm-z/gperftools.git

1. Configure and build the software (you may want to choose a different installation directory instead of /usr, e.g. /opt/gperftools):

        ./autogen.sh
        ./configure --prefix /usr
        make

1. **(Optional)** Run unit tests:

        make check
      
1. Install gperftools under /usr:

        sudo make install

   This will install the `pprof` command under /usr/bin, and libraries such as libprofiler.so and libtcmalloc.so under /usr/lib. For more information on using these performance tools in your programs, refer to the [gperftools documentation](https://github.com/gperftools/gperftools/wiki).