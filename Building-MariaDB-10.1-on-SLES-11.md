# Building MariaDB 10.1
[MariaDB](https://mariadb.org/) is already available for Linux on z Systems from the official RHEL 7.1 and SLES 12 repositories, but it is not available for RHEL 6 or SLES 11 SP3. Moreover, RHEL 7.1 ships only MariaDB 5.5, rather than the latest stable branch, 10.1. This article describes the procedure for building and configuring MariaDB Galera Cluster 10.1 on Linux on z Systems. The instructions have been tested on SLES 11 SP3.

_**General Notes:**_ 	 
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory  `/<source_root>/`  will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

_**Note:** At the time of writing the recipe the version was 10.1.11._
 
### Building MariaDB 10.1
1. Install the prerequisites for building the code
    ```
      sudo zypper install -y  git wget tar patch openssl-devel boost-devel check-devel gcc make gcc-c++ ncurses-devel bison libxml2-devel libcurl-devel libexpat-devel libarchive-devel glib-devel fdupes 
    ```

2. Building MariaDB requires CMake 2.6.3 or newer. Run the following commands to build and install CMake 2.8.11:

        cd /<source_root>/
        wget http://download.opensuse.org/repositories/openSUSE:/13.1/standard/src/cmake-2.8.11.2-4.1.2.src.rpm
        rpmbuild --rebuild cmake-2.8.11.2-4.1.2.src.rpm
        sudo rpm -i /usr/src/packages/RPMS/s390x/cmake-2.8.11.2-4.1.2.s390x.rpm

3. MariaDB uses [jemalloc](http://www.canonware.com/jemalloc/), a scalable concurrent memory allocator. This needs to be built first:

        cd /<source_root>/
        wget -O jemalloc  https://github.com/jemalloc/jemalloc/releases/download/3.6.0/jemalloc-3.6.0.tar.bz2
        tar xjf jemalloc
        cd jemalloc-3.6.0
        ./configure
        make
        sudo make install
        sudo /sbin/ldconfig -v

    `make install`  will place libjemalloc.so.1, libjemalloc.a and libjemalloc_pic.a in /usr/local/lib/, and jemalloc.h in /usr/local/include/jemalloc/

4. Get the source code of MariaDB:

        cd /<source_root>/
        git clone -b 10.1 https://github.com/MariaDB/server.git
        cd server
        BUILD/autorun.sh
        ./configure
        

5. Build the code and package the binaries into a tarball:
    
        make
		sudo make install
        sudo make package
        sudo cp mariadb-10.1.11-linux-s390x.tar.gz /opt/

    The  `make package`  command takes a while to complete; it might seem to hang but it will finish, and generate a tarball named mariadb-10.1.11-linux-s390x.tar.gz.

    _**Note:** Above command may give `cannot stat mariadb-10.1.11-linux-s390x.tar.gz: No such file or directory` as name of this tarball may have been changed with its version. Please check tarball name using `ls` command and modify above command accordingly. Same applies to later commands too._  

6. To install MariaDB Galera Cluster, simply unpack the tarball in the installation directory prefix:

        cd /opt
        sudo tar xzvf mariadb-10.1.11-linux-s390x.tar.gz

    This will create the directory mariadb-10.1.11-linux-s390x/ under /opt/, and install all programs, data files and scripts underneath it.
    
    When installing the MariaDB package on a different system which runs the same version of Linux, the dependencies must also be installed (libjemalloc.so.1 should be copied from the original build system):
    
        sudo /sbin/ldconfig -v

### Configuring and Running MariaDB
1. Copy the sample configuration file to /etc/ and rename it to my.cnf:
        
		cd /<source_root>/
        sudo cp /opt/mariadb-10.1.11-linux-s390x/support-files/my-innodb-heavy-4G.cnf /etc/my.cnf
    Edit my.cnf to configure MariaDB according to your needs. Refer to the [configuration help](https://mariadb.com/kb/en/mariadb/mysqld-configuration-files-and-groups/) and the [full option list](https://mariadb.com/kb/en/mariadb/mysqld-options/) for more information.

2. Before starting the server, created and populated the data directory. The default location for the data directory is /opt/mariadb--10.1.10-linux-s390x/data/ but it can be changed with the  `--datadir`  command-line option. You might also want to create a non-privileged user named "mysql" for running the server, instead of running it as root. For example, you could issue following commands to create the user and initialize the data directory:

        cd /<source_root>/
		sudo /usr/sbin/useradd -m mysql
        sudo mkdir /home/mysql/data
        sudo chown mysql /home/mysql/data
        cd /opt/mariadb-10.1.11-linux-s390x
        sudo scripts/mysql_install_db --user=mysql --basedir=/opt/mariadb-10.1.11-linux-s390x --datadir=/home/mysql/data

    When this script finishes, it will print additional instructions to set the password for the MariaDB root user, and for securing a MariaDB installation.
    
3. To start the server, run:

        cd /opt/mariadb-10.1.11-linux-s390x
        sudo bin/mysqld_safe --user=mysql --datadir=/home/mysql/data &
        
    To shut down the server, send SIGTERM to its process ID, e.g. (replace "localhost" with the actual hostname)
    
        sudo kill -TERM `cat /home/mysql/data/localhost.pid`

### Building the Galera wsrep provider
The Galera wsrep ("write set replication") provider is a library that extends a number of database products (including MariaDB) with replication capabilities. We have patched it so that it can be built on Linux on z Systems. It will need to be installed before MariaDB clustering can be enabled.

1. Building Galera requires SCons. Download the SCons RPM and install it:

        cd <source-root> 
		wget http://downloads.sourceforge.net/project/scons/scons/2.3.4/scons-2.3.4-1.noarch.rpm
        sudo rpm -i scons-2.3.4-1.noarch.rpm

2. Build gcc-4.8.3 from source using below steps:

        cd /<source_root>/
        wget https://ftp.gnu.org/gnu/gcc/gcc-4.8.3/gcc-4.8.3.tar.gz
        tar xzf gcc-4.8.3.tar.gz
        cd gcc-4.8.3
        ./contrib/download_prerequisites
        cd ..
        mkdir objdir
        cd objdir
        $PWD/../gcc-4.8.3/configure --prefix=$HOME/gcc-4.8.3 --enable-languages=c,c++,fortran,go --disable-multilib
        make 
		sudo make install

		
	Commands to set gcc-4.8.3 as default gcc:

		sudo update-alternatives --install /usr/bin/gcc gcc /<source_root>/gcc-4.8.3/bin/gcc 50
		sudo update-alternatives --install /usr/bin/g++ g++ /<source_root>/gcc-4.8.3/bin/g++ 50
		sudo update-alternatives --install /usr/bin/cpp cpp /<source_root>/gcc-4.8.3/bin/cpp 50
		sudo update-alternatives --install /usr/bin/c++ c++ /<source_root>/gcc-4.8.3/bin/c++ 50
		export LD_LIBRARY_PATH=/<source_root>/gcc-4.8.3/lib64   

3. Clone the Galera source code from the [official GitHub repository](https://github.com/codership/galera), then check out the 25.3.10 branch:

        cd /<source_root>/
        git clone https://github.com/codership/galera.git
        cd galera
        git checkout 25.3.10
        
4. Patch the Galera code so that it will build on z. Edit the file SConstruct, and replace the  `if... elif... else`  block on lines 90 to lines 108 with following:

        compile_arch = ' -march=z900'
        link_arch    = ''
        x86 = 0

    Then edit chromium/build_config.h, and add following text after line 127:
    
        #elif defined(__s390__)
        #if defined(__s390x__)
        #define ARCH_CPU_64_BITS 1
        #else
        #define ARCH_CPU_32_BITS 1
        #endif
        #define ARCH_CPU_BIG_ENDIAN 1
    
5. Edit asio/asio/detail/fenced_block.hpp, and comment these three lines 22-24:

        // #if !defined(BOOST_HAS_THREADS)
        // #error "BOOST_HAS_THREADS not defined"
        // #endif

6. Now issue following command to build the code:

        scons strict_build_flags=0 --config=force
		
	_**Note:**_ While running scons command if you get ` g++: internal compiler error: Killed (program cc1plus)` you need to increase free space in your machine. 
    
7. Issue following commands to install the Galera arbitration daemon (garbd) and the wsrep provider library:

        sudo cp garb/garbd /usr/local/sbin/        
        sudo cp libgalera_smm.so /usr/local/lib64/
        sudo /sbin/ldconfig -v

8. The MariaDB configuration file  '/server/support-files/my-innodb-heavy-4G.cnf' needs to be updated as well. The exact configuration values will be different for each installation. Following is an example for a two-node cluster:

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

    
    MariaDB is now ready for clustering. Refer to following tutorials and documentation for more information on setting up and configuring a Galera cluster:
    
* [Blog: Setting up a MySQL cluster with MariaDB Galera](http://jmoses.co/2014/03/18/setting-up-a-mysql-cluster-with-mariadb-galera.html)
* [Blog: Getting started with MariaDB Galera Cluster and Percona XtraDB Cluster](http://blog.yannickjaquier.com/mysql/getting-started-with-mariadb-galera-cluster-and-percona-xtradb-cluster.html) 
* [MariaDB documentation ](https://mariadb.com/kb/en/mariadb/getting-started-with-mariadb-galera-cluster/)
* [Galera documentation](http://galeracluster.com/documentation-webpages/dbconfiguration.html)
