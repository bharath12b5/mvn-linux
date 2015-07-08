[MariaDB](https://mariadb.org/) is already available for Linux on z Systems from the official RHEL 7.1 and SLES 12 repositories, but it is not available for RHEL 6 or SLES 11 SP3. Moreover, RHEL 7.1 ships only MariaDB 5.5, rather than the latest stable branch, 10.0. This article describes the procedure for building and configuring MariaDB Galera Cluster 10.0.19 on Linux on z Systems. The instructions have been tested on SLES 11 SP3.

## Building MariaDB

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
        cmake -DWITH_WSREP=ON -DWITH_INNODB_DISALLOW_WRITES=1 -DWITH_JEMALLOC=yes -DCMAKE_INSTALL_PREFIX=/opt/mariadb-galera-10.0.19-linux-s390x .

   The installation directory /opt/mariadb-galera-10.0.19-linux-s390x/ could be placed in another location, e.g. /usr/, if desired, but it is recommended to leave the base directory name as "mariadb-galera-10.0.19-linux-s390x", since this name will be used in the packaging step, below.

5. Build the code and package the binaries into a tarball:

        make
        sudo make package

   The `make package` command takes a while to complete; it might seem to hang but it will finish, and generate a tarball named mariadb-galera-10.0.19-linux-s390x.tar.gz.

6. To install MariaDB Galera Cluster, simply unpack the tarball in the installation directory prefix:

        cd /opt
        sudo tar xzvf ~/mariadb-10.0.19/mariadb-galera-10.0.19-linux-s390x.tar.gz

   This will create the directory mariadb-galera-10.0.19-linux-s390x/ under /opt/, and install all programs, data files and scripts underneath it.

   When installing the MariaDB package on a different system which runs the same version of Linux, the dependencies must also be installed (libjemalloc.so.1 should be copied from the original build system):

        sudo zypper install libxml2
        sudo cp ~/libjemalloc.so.1 /usr/local/lib/
        sudo /sbin/ldconfig -v

## Configuring and Running MariaDB

1. Copy the sample configuration file to /etc/ and rename it to my.cnf:

        sudo cp /opt/mariadb-galera-10.0.19-linux-s390x/support-files/my-innodb-heavy-4G.cnf /etc/my.cnf

   Edit my.cnf to configure MariaDB according to your needs. Refer to the [configuration help](https://mariadb.com/kb/en/mariadb/mysqld-configuration-files-and-groups/) and the [full option list](https://mariadb.com/kb/en/mariadb/mysqld-options/) for more information.

2. Before starting the server, the data directory need to be created and populated. The default location for the data directory is /opt/mariadb-galera-10.0.19-linux-s390x/data/ but it can be changed with the `--datadir` command-line option. You might also want to create a non-privileged user named "mysql" for running the server, instead of running it as root. For example, you could issue the following commands to create the user and initialize the data directory:

        sudo /usr/sbin/useradd -m mysql
        sudo mkdir /home/mysql/data
        sudo chown mysql /home/mysql/data
        cd /opt/mariadb-galera-10.0.19-linux-s390x
        sudo scripts/mysql_install_db --user=mysql --basedir=/opt/mariadb-galera-10.0.19-linux-s390x --datadir=/home/mysql/data

   When this script finishes, it will print additional instructions to set the password for the MariaDB root user, and for securing a MariaDB installation.

3. To start the server, run:

        cd /opt/mariadb-galera-10.0.19-linux-s390x
        sudo bin/mysqld_safe --user=mysql --datadir=/home/mysql/data

   To shut down the server, send SIGTERM to its process ID, e.g. (replace "localhost" with the actual hostname)

        sudo kill -TERM `cat /home/mysql/data/localhost.pid`

## Building the Galera wsrep provider

The Galera wsrep ("write set replication") provider is a library that extends a number of database products (including MariaDB) with replication capabilities. We have patched it so that it can be built on Linux on z Systems. It will need to be installed before MariaDB clustering can be enabled.

1. Building Galera requires SCons. Download the SCons RPM and install it:

        wget http://downloads.sourceforge.net/project/scons/scons/2.3.4/scons-2.3.4-1.noarch.rpm
        sudo rpm -i scons-2.3.4-1.noarch.rpm

   On SLES 12, SCons is available in the official repository, in which case you could issue this command instead:

        sudo zypper install scons

2. Galera depends on several other libraries. Fortunately, SLES 11 SP3 provides all of them:

        sudo zypper install boost-devel check-devel openssl-devel

3. Clone the Galera source code from our GitHub repository, then check out the 25.3.10 branch and build it:

        git clone https://github.com/linux-on-ibm-z/galera.git
        cd galera
        git checkout 25.3.10
        scons strict_build_flags=0

   This will create libgalera_smm.so in the build directory.

4. Issue the following commands to install the Galera arbitration daemon (garbd) and the wsrep provider library:

        sudo cp garb/garbd /usr/local/sbin/        
        sudo cp libgalera_smm.so /usr/local/lib64/
        sudo /sbin/ldconfig -v

5. The MariaDB configuration file needs to be updated as well. The exact configuration values will be different for each installation; the following is an example for a two-node cluster:

        [mysqld]
        binlog_format=row
        innodb_autoinc_lock_mode=2
        query_cache_size=0
        
        wsrep_provider=/usr/local/lib64/libgalera_smm.so
        wsrep_cluster_address='gcomm://192.168.0.101,192.168.0.102'
        wsrep_cluster_name='mariacluster'
        wsrep_node_address='192.168.0.101'
        wsrep_node_name='dbnode1'
        wsrep_sst_method=rsync
        wsrep_sst_auth=root:rootpw

   MariaDB is now ready for clustering. Refer to the following tutorials and documentation for more information on setting up and configuring a Galera cluster:

    - [Blog: Setting up a MySQL cluster with MariaDB Galera](http://jmoses.co/2014/03/18/setting-up-a-mysql-cluster-with-mariadb-galera.html)
    - [Blog: Getting started with MariaDB Galera Cluster and Percona XtraDB Cluster](http://blog.yannickjaquier.com/mysql/getting-started-with-mariadb-galera-cluster-and-percona-xtradb-cluster.html)
    - [MariaDB documentation](https://mariadb.com/kb/en/mariadb/getting-started-with-mariadb-galera-cluster/) 
    - [Galera documentation](http://galeracluster.com/documentation-webpages/dbconfiguration.html)