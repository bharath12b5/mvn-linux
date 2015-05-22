# Building MySQL

The following build instructions have been tested on RHEL7 and SLES12 on IBM z Systems.

### Version
>5.6.24

####1.  Install the following dependencies:
    git
    gcc
    gcc-c++
    make
    cmake
    bison
    ncurses-devel
    perl-Data-Dumper (on rhel7)
    
##### Example:

###### RHEL7 :
     yum install -y git gcc gcc-c++ make cmake bison ncurses-devel perl-Data-Dumper

###### SLES12:
     zypper install -y git gcc gcc-c++ make cmake bison ncurses-devel

####2.  Clone MYSQL git repository:
    git clone https://github.com/mysql/mysql-server.git

####3. Checkout branch 5.6:
     cd mysql-server
     git branch
     git checkout 5.6

####4. Configure, build and install:
     cmake .
     make
     make install

####5. Run test cases:
     make test

####6. Post installation setup and testing:

#### create a user and group with name mysql
     useradd mysql
     Note: If group "mysql" does not exist, then create using the following command
     groupadd mysql
     
#### Intializing data directory:
     cd /usr/local/mysql (mysql installation dir)
     chown -R mysql .
     chgrp -R mysql .
     scripts/mysql_install_db --user=mysql

#### Starting the server:
     bin/mysqld_safe --user=mysql &

#### Testing the server:
     bin/mysqladmin version 

#### To verify shutdown:
     bin/mysqladmin -u root shutdown

#### To start and stop server as init.d service:
     cp support-files/mysql.server /etc/init.d/mysql
     /etc/init.d/mysql start
     /etc/init.d/mysql stop

#### To see existing databases:
     bin/mysqlshow
     Note:
			1. To execute bin/mysql commands, mysql server needs to be started.
			2. If password is set for root user then use bin/mysqladmin and bin/mysql commands with -u and -p options. 
				For Example:# bin/mysqladmin -u root -p version
				Enter password: (Type password here or press enter for blank password)
