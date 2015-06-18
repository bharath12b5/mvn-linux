[MariaDB](https://mariadb.org/) is already available for Linux on z Systems from the official RHEL 7.1 and SLES 12 repositories, but it is not available for RHEL 6 or SLES 11 SP3. Moreover, RHEL 7.1 ships only MariaDB 5.5, rather than the latest stable branch, 10.0. This article describes the procedure for building MariaDB Galera Cluster 10.0.19 on Linux on z Systems. The instructions have been tested on SLES 11 SP3.

1. Install the prerequisites:

        zypper install cmake gcc-c++ libxml2-devel

2. MariaDB uses [jemalloc](http://www.canonware.com/jemalloc/), a scalable concurrent memory allocator. This needs to be built first:
        
        wget https://github.com/jemalloc/jemalloc/releases/download/3.6.0/jemalloc-3.6.0.tar.bz2
        tar xjf jemalloc-3.6.0.tar.bz2
        cd jemalloc-3.6.0
        ./configure
        make
        sudo make install

   `make install` will place libjemalloc.so.1, libjemalloc.a and libjemalloc_pic.a in /usr/local/lib/, and jemalloc.h in /usr/local/include/jemalloc/. Run `ldconfig -v` as root to update the dynamic linker.

3. Download the source code (mariadb-galera-10.0.19.tar.gz) for MariaDB Galera Cluster 10.0.19 from [mariadb.org](https://downloads.mariadb.org/mariadb-galera/10.0.19/).

4. Unpack the source code and configure it with CMake:

        tar xzf mariadb-galera-10.0.19.tar.gz
        cd mariadb-10.0.19
        cmake -DWITH_WSREP=ON -DWITH_INNODB_DISALLOW_WRITES=1 -DWITH_JEMALLOC=yes -DCMAKE_INSTALL_PREFIX=/usr/local .

   The installation location /usr/local/ could be replaced with another directory (e.g. /opt/mariadb/), if desired.

5. Build the code and install it:

        make
        sudo make install

6. Run this command to create a binary package:

        sudo make package

   The `make package` command takes a while to complete; it might seem to hang but it will finish, and generate a tarball named mariadb-galera-10.0.19-linux-s390x.tar.gz. To install MariaDB Galera Cluster on a different system, copy the tarball and /usr/local/lib/libjemalloc.so.1 to the target system. Place libjemalloc.so.1 in /usr/local/lib/ and run  `ldconfig -v`. Run `zypper install libxml2` to install the prerequisite. Finally, unpack the tarball in a suitable location; MariaDB is now ready to run.

7. Refer to the very good [MariaDB documentation](https://mariadb.com/kb/en/mariadb/getting-started-with-mariadb-galera-cluster/) for more information on how to set up and configure a database cluster.