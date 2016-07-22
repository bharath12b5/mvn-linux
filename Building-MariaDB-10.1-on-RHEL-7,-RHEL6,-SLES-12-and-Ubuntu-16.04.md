# Building MariaDB

Below versions of MariaDB are available in respective distributions at the time of this recipe creation:

*    RHEL   7     has `5.5.47`
*    SLES   12    has `10.0.25-20.6.1`
*    Ubuntu 16.04 has `10.0.25`

The instructions provided below specify the steps to build MariaDB v10.1 on Linux on the IBM z Systems for RHEL 6/7, SLES 12 and Ubuntu 16.04.

_**General Notes:**_ 	 
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory  `/<source_root>/`  will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._


### Section 1: Install following dependencies

*   RHEL7/RHEL6: 
    ```
        sudo yum install -y git wget cmake gcc gcc-c++ make ncurses-devel bison tar boost-devel check-devel openssl-devel perl-CPAN 'perl(Test::More)'
                      
    ```

*   SLES12:
    ```
        sudo zypper install -y git wget tar cmake gcc gcc-c++ make ncurses-devel boost-devel check-devel openssl-devel bison scons
 
    ```

*   Ubuntu 16.04

      _**Note:** Add following repos in `/etc/apt/sources.list` file and upgrade the system, if any of the mentioned
      package on Ubuntu is missing._ 
    ```shell
        deb http://ports.ubuntu.com/ubuntu-ports/ xenial main restricted universe multiverse  
        deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial main restricted universe multiverse  
        deb http://ports.ubuntu.com/ubuntu-ports/ xenial-security main restricted universe multiverse  
        deb http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main restricted universe multiverse  
        deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-security main restricted universe multiverse  
        deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main restricted universe multiverse

    ```
    ```
        sudo apt-get update
        sudo apt-get install -y build-essential libncurses-dev git wget cmake gcc make tar libpcre3-dev bison scons libboost-dev libboost-program-options-dev openssl dh-autoreconf libssl-dev
                      
    ```

### Section 2: Build and Install
1. Get MariaDB source

        cd /<source_root>/
        git clone -b 10.1 https://github.com/MariaDB/server.git
        cd server 
        
2. Build and install the source
 *  For RHEL/SLES12
     ```
    BUILD/autorun.sh 
    ./configure 
    make 
    sudo make install 
     ```

 *   For Ubuntu 16.04
     ```
     BUILD/autorun.sh 
    ./configure --WITH_SAFEMALLOC=ON --with-debug
     make 
     sudo make install 
     ```
3. Execute test cases

        make test
        
4. Create a user and group with name mysql

        sudo useradd mysql
        sudo groupadd mysql

5. Give owner permission to the mysql directory

        cd /usr/local/mysql		
        sudo chmod -R o+rwx .

6. Create database and populate test data into database

        sudo scripts/mysql_install_db --user=mysql

7. Copy script to start and stop MySql server 

        sudo cp support-files/mysql.server /etc/init.d/mysql

8. Start MySQL server

        sudo /etc/init.d/mysql start
		
		Note: If you get D-bus error inside docker container, run below commands to start MySQL server
		
		sudo chown -R mysql .
		sudo chgrp -R mysql .
		/etc/init.d/mysql start
		
9. Display version

        sudo bin/mysqladmin version --user=mysql

### Section 3: Building the Galera wsrep provider
 The Galera wsrep ("write set replication") provider is a library that extends a number of database products (including MariaDB) with replication capabilities. We have patched it so that it can be built on Linux on z Systems. It will need to be installed before MariaDB clustering can be enabled.
 
1. Building Galera requires SCons. Download the SCons RPM and install it:(Only for **RHEL7/RHEL6**)

    ```
        cd /<source_root>/
        wget http://downloads.sourceforge.net/project/scons/scons/2.3.4/scons-2.3.4-1.noarch.rpm
        sudo rpm -i scons-2.3.4-1.noarch.rpm
    ``` 
2. Install the following dependencies required for galera:(Only for **Ubuntu 16.04**)
 *   Install makeinfo

    ```
        cd /<source_root>/
        wget http://ftp.gnu.org/gnu/texinfo/texinfo-5.0.tar.gz
        tar -xvf texinfo-5.0.tar.gz
        cd texinfo-5.0
        ./configure --prefix=/usr
        make
        sudo make install
    ``` 

 *   Install check dependency

    ```
        cd /<source_root>/
        sudo apt-get install libsubunit-dev libsubunit0 pkg-config 
        git clone https://github.com/libcheck/check
        cd check/
        git checkout 0.10.0
        autoreconf --install
        export LD_LIBRARY_PATH=/usr/lib/s390x-linux-gnu/openssl-1.0.0/engines:/usr/lib/s390x-linux-gnu/:/usr/local/lib/
        ./configure
        make
        sudo make install
    ``` 

3. Clone the Galera source code from the official GitHub repository, then check out the 25.3.10 branch:

    ```
        cd /<source_root>/
        git clone https://github.com/codership/galera.git
        cd galera
        git checkout 25.3.10
    ```
    
4. Patch the Galera code so that it will build on z. Edit the file SConstruct, and replace the  `if... elif... else`  block on lines 90 to lines 108 with following:
 *  For RHEL7/SLES12/Ubuntu 16.04
     ```
        compile_arch = ' -march=z196 -mtune=zEC12'
        link_arch    = ''
        x86 = 0
     ```

 *   For RHEL6
     ```
        compile_arch = ' -march=z196 '
        link_arch    = ''
        x86 = 0
     ```

     Then edit chromium/build_config.h, and add following text after line 127:(For RHEL6/RHEL7/SLES12/Ubuntu 16.04
)
    
     ```
        #elif defined(__s390__)
        #if defined(__s390x__)
        #define ARCH_CPU_64_BITS 1
        #else
        #define ARCH_CPU_32_BITS 1
        #endif
        #define ARCH_CPU_BIG_ENDIAN 1
     ```
    
5. Now issue following command to build the code:

    ```
        scons strict_build_flags=0 --config=force
    ```

    _**Note:** If the following build error observed then increase the time out period in a function `tcase_set_timeout(tc, 120)` (change 120 to a value either 480 or 600) present in the file 
`galera/galera/tests/saved_state_check.cpp` and run the above command again._    

    ```
        33%: Checks: 3, Failures: 1, Errors: 1
        galera/tests/saved_state_check.cpp:121:E:saved_state:test_unsafe:0: (after this point) Test timeout expired
        galera/tests/saved_state_check.cpp:155:F:saved_state:test_corrupt:0: Failure 'uuid == WSREP_UUID_UNDEFINED' occurred

    ```

6. Issue following commands to install the Galera arbitration daemon (garbd) and the wsrep provider library:

    ```
        sudo cp garb/garbd /usr/local/sbin/        
        sudo cp libgalera_smm.so /usr/local/lib64/
        sudo /sbin/ldconfig -v
    ```
 _**Note:** For Ubuntu 16.04, if `/usr/local/lib64/` path does not exist then using `mkdir -p /usr/local/lib64/` command create the path and copy `libgalera_smm.so`._

7. The MariaDB configuration file  '/server/support-files/my-innodb-heavy-4G.cnf' needs to be updated as well. The exact configuration values will be different for each installation. Following is an example for a two-node cluster:

    ```
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
    ```
    
    MariaDB is now ready for clustering. Refer to following tutorials and documentation for more information on setting up and configuring a Galera cluster:
    
* [Blog: Setting up a MySQL cluster with MariaDB Galera](http://jmoses.co/2014/03/18/setting-up-a-mysql-cluster-with-mariadb-galera.html)
* [Blog: Getting started with MariaDB Galera Cluster and Percona XtraDB Cluster](http://blog.yannickjaquier.com/mysql/getting-started-with-mariadb-galera-cluster-and-percona-xtradb-cluster.html) 
* [MariaDB documentation ](https://mariadb.com/kb/en/mariadb/getting-started-with-mariadb-galera-cluster/)
* [Galera documentation](http://galeracluster.com/documentation-webpages/dbconfiguration.html)


