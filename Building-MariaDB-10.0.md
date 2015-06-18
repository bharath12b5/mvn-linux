[MariaDB](https://mariadb.org/) is already available for Linux on z Systems from the official RHEL 7.1 and SLES 12 repositories, but it is not available for RHEL 6 or SLES 11 SP3. Moreover, RHEL 7.1 ships only MariaDB 5.5, rather than the latest stable branch, 10.0. This article describes the procedure for building MariaDB Galera Cluster 10.0.19 on Linux on z Systems. The instructions have been tested on RHEL 6 and SLES 11 SP3.

1. Install the prerequisites for building the code:

        sudo zypper install gcc-c++ libxml2-devel

2. Building MariaDB requires CMake 2.6.3 or newer, but SLES 11 SP3 only provides CMake 2.6.2. On SLES 11 SP3, run the following commands to build and install CMake 2.8.11:

        sudo zypper install fdupes libarchive-devel libcurl-devel libexpat-devel
        wget http://download.opensuse.org/repositories/openSUSE:/13.1/standard/src/cmake-2.8.11.2-4.1.2.src.rpm
        rpmbuild --rebuild cmake-2.8.11.2-4.1.2.src.rpm
        sudo rpm -i ~/rpmbuild/RPMS/s390x/cmake-2.8.11.2-4.1.2.s390x.rpm

   On other versions of Linux, just install CMake as usual:

        sudo zypper install cmake

3. MariaDB uses [jemalloc](http://www.canonware.com/jemalloc/), a scalable concurrent memory allocator. This needs to be built first:
        
        wget https://github.com/jemalloc/jemalloc/releases/download/3.6.0/jemalloc-3.6.0.tar.bz2
        tar xjf jemalloc-3.6.0.tar.bz2
        cd jemalloc-3.6.0
        ./configure
        make
        sudo make install
        sudo /sbin/ldconfig -v

   `make install` will place libjemalloc.so.1, libjemalloc.a and libjemalloc_pic.a in /usr/local/lib/, and jemalloc.h in /usr/local/include/jemalloc/.

4. Download the source code (mariadb-galera-10.0.19.tar.gz) for MariaDB Galera Cluster 10.0.19 from [mariadb.org](https://downloads.mariadb.org/mariadb-galera/10.0.19/). Unpack the source code and configure it with CMake as follows:

        tar xzf mariadb-galera-10.0.19.tar.gz
        cd mariadb-10.0.19
        cmake -DWITH_WSREP=ON -DWITH_INNODB_DISALLOW_WRITES=1 -DWITH_JEMALLOC=yes -DCMAKE_INSTALL_PREFIX=/usr/local .

   The installation location /usr/local/ could be replaced with another directory (e.g. /opt/mariadb/), if desired.

5. Build the code and install it on the same system:

        make
        sudo make install

   MariaDB is now ready to run.

6. **(Optional)** Run the following command to create a binary package that can be installed on a different system:

        sudo make package

   The `make package` command takes a while to complete; it might seem to hang but it will finish, and generate a tarball named mariadb-galera-10.0.19-linux-s390x.tar.gz. To install MariaDB Galera Cluster on a different system (that runs the same Linux version), copy the tarball and /usr/local/lib/libjemalloc.so.1 to the target system. Place libjemalloc.so.1 in /usr/local/lib/ and run  `/sbin/ldconfig -v` as root. Run `zypper install libxml2` to install the prerequisite. Finally, unpack the tarball in a suitable location, e.g. /usr/local/; MariaDB is now ready to run.

7. Refer to the very good [MariaDB documentation](https://mariadb.com/kb/en/mariadb/getting-started-with-mariadb-galera-cluster/) for more information on how to set up and configure a database cluster.