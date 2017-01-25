<!---PACKAGE:Joomla--->
<!---DISTRO:SLES 12:3.6.x--->
<!---DISTRO:SLES 11:3.6.x--->
<!---DISTRO:RHEL 7.1:3.6.x--->
<!---DISTRO:RHEL 6.6:3.6.x--->
<!---DISTRO:Ubuntu 16.x:3.6.x--->


The instructions provided below specify the steps to build Joomla 3.6.4  on the IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.


_**General Notes:**_

* _When following the steps below, please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions.  This is a temporary writeable directory anywhere you'd like to place it._

## Building Joomla

### Section 1: Install dependencies

* RHEL 6.8
```
sudo yum install -y httpd mysql mysql-server wget libjpeg-devel libpng-devel unzip tar gcc gcc-c++ make cmake httpd-devel libxml2 libxml2-devel
```

* RHEL 7.1/7.2/7.3
```
sudo yum install -y wget httpd httpd-devel.s390x mariadb mariadb-server unzip tar make libxml2 libxml2-devel libpng libpng-devel gcc-c++
```
 
* SLES 11-SP4
```
sudo zypper install -y apache2 apache2-devel tar wget gcc gcc-c++ libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel mysql libjpeg-devel libpng-devel unzip
```
	 
* SLES 12/12-SP1/12-SP2
```
sudo zypper install -y apache2 apache2-devel tar wget mariadb gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel wget libjpeg-devel libpng-devel unzip
```
* Ubuntu 16.04/16.10
```
sudo apt-get update
sudo apt-get install -y mysql-server
sudo apt-get install -y wget apache2 php7.0 php7.0-mysql php7.0-mcrypt libapache2-mod-php7.0 libapache2-mod-perl2  php7.0-cli php7.0-common php7.0-curl php7.0-dev unzip curl  
```

#####Create the `/<source_root>/` directory as mentioned above
```
mkdir /<source_root>/
```


####Build and Install libmcrypt (For RHEL and SLES)

```bash
cd /<source_root>/
wget http://sourceforge.net/projects/mcrypt/files/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz
tar -xvzf libmcrypt-2.5.8.tar.gz
cd libmcrypt-2.5.8
./configure
make
sudo make install
```

####Build and Install PHP (For RHEL and SLES)

1 . Download PHP source code

```bash
cd /<source_root>/
wget http://www.php.net/distributions/php-5.6.8.tar.gz 
tar xvzf php-5.6.8.tar.gz
```

2 . Configure

* RHEL 6.8
```bash
cd /<source_root>/php-5.6.8
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php --with-mysql --with-gd --with-zlib --with-mcrypt --with-apxs2=/usr/sbin/apxs
```

* RHEL 7.1/7.2/7.3
```bash
cd /<source_root>/php-5.6.8
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php --with-mysql --with-gd --with-zlib --with-mcrypt --with-apxs2=/usr/bin/apxs
```

* SLES 11-SP4 and SLES 12/12-SP1/12-SP2
```bash
cd /<source_root>/php-5.6.8
./configure --prefix=/usr/local/php --with-apxs2=/usr/sbin/apxs2 --with-config-file-path=/usr/local/php --with-mysql --with-gd --with-zlib  --with-mcrypt 
```

3 . Build PHP
```bash
cd /<source_root>/php-5.6.8
make  
sudo make install
export PATH=/usr/local/php/bin/:$PATH       
sudo ln -s -f /usr/local/php/bin/phar.phar /usr/local/php/bin/phar (For RHEL 7.1/7.2/7.3)
```

###Section 2: Make Changes in the configuration files

_(You should be root to do the below changes.)_


* RHEL 6.8 and RHEL 7.1/7.2/7.3

  * `/etc/httpd/conf/httpd.conf`

     ```diff
     @@ -117,7 +116,7 @@ 
      # documents. By default, all requests are taken from this directory, but
      # symbolic links and aliases may be used to point to other locations.
      #
     -DocumentRoot "/var/www/html/Joomla"
     +DocumentRoot "/var/www/html"

      #
      # Relax access to content within /var/www.
     @@ -129,7 +128,7 @@
      </Directory>

      # Further relax access to the default document root:
     -<Directory "/var/www/html/Joomla">
     +<Directory "/var/www/html">

     @@ -352,4 +352,9 @@ 
      #
      # Load config files in the "/etc/httpd/conf.d" directory, if any.
       IncludeOptional conf.d/*.conf
     + AddType application/x-httpd-php .php
     +
     + <Directory />
     + DirectoryIndex index.php
     + </Directory>

    ```
     
* SLES 11-SP4 and SLES 12/12-SP1/12-SP2

  * `/etc/apache2/default-server.conf` 
    ```diff
    @@ -3,12 +3,12 @@
     # deleted here, or overriden elswhere.
     #
    
    -DocumentRoot "/srv/www/htdocs"
    +DocumentRoot "/srv/www/htdocs/Joomla"

     #
     # Configure the DocumentRoot
     #
    -<Directory "/srv/www/htdocs">
    +<Directory "/srv/www/htdocs/Joomla">
            # Possible values for the Options directive are "None", "All",
            # or any combination of:
            #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
    ```

  * `/etc/apache2/httpd.conf`
    * SLES 11-SP4 and SLES 12
      ```diff
      @@ -204,7 +204,6 @@
       # The file below is generated from /etc/sysconfig/apache2,
       # include arbitrary files as named in APACHE_CONF_INCLUDE_FILES and
       # APACHE_CONF_INCLUDE_DIRS
      -Include /etc/apache2/sysconfig.d/include.conf


       ### Virtual server configuration ############################################
      @@ -229,3 +228,8 @@
       #       putting its name into APACHE_CONF_INCLUDE_FILES in
       #       /etc/sysconfig/apache2 -- this will make system updates
       #       easier :)
      +AddType application/x-httpd-php .php
      +<Directory />
      + DirectoryIndex index.php
      +</Directory>
      +LoadModule php5_module /usr/lib64/apache2/libphp5.so
      ```

    * SLES 12-SP1/12-SP2
      ```diff
      @@ -226,3 +226,8 @@
       #       putting its name into APACHE_CONF_INCLUDE_FILES in
       #       /etc/sysconfig/apache2 -- this will make system updates
       #       easier :)
      +AddType application/x-httpd-php .php
      +<Directory />
      + DirectoryIndex index.php
      +</Directory>
      +LoadModule php5_module /usr/lib64/apache2/libphp5.so
      ```

### Section 3: Download Joomla source code and configure

1 . Download and install Joomla

* RHEL 6.8, RHEL 7.1/7.2/7.3 and Ubuntu 16.04/16.10
 ```bash
  cd /var/www/html
  sudo mkdir Joomla
  cd Joomla
  sudo wget https://github.com/joomla/joomla-cms/releases/download/3.6.4/Joomla_3.6.4-Stable-Full_Package.zip
  sudo unzip Joomla_3.6.4-Stable-Full_Package.zip
  sudo chmod a+w /var/www/html/Joomla/
```

* SLES 11-SP4 and SLES 12/12-SP1/12-SP2:
 ```bash	
 cd /srv/www/htdocs
 sudo mkdir Joomla
 cd Joomla
 sudo wget -O Joomla_3.6.4-Stable-Full_Package.zip  https://github.com/joomla/joomla-cms/releases/download/3.6.4/Joomla_3.6.4-Stable-Full_Package.zip
 sudo unzip Joomla_3.6.4-Stable-Full_Package.zip
 sudo chmod a+w /srv/www/htdocs/Joomla/
```


2 . Rename configuration.php-dist as configuration.php 
 ```bash
 sudo mv installation/configuration.php-dist installation/configuration.php 
 ```

### Section 4: Start Apache HTTP Server and MySQL Sever

1. Initialize MySQL server 

```bash
 sudo /usr/bin/mysql_install_db --user=mysql  (For RHEL and SLES)
 sudo /usr/sbin/mysqld --initialize --user=mysql --datadir=/var/lib/mysql/data  (For Ubuntu)
```

2 . Start MySQL server
 ```
 sudo mkdir -p /var/log/mysql (For SLES 11 and SLES 12)
 sudo /usr/bin/mysqld_safe --user=mysql &
 ```

3 . Grant privileges
 
```bash
	sudo /usr/bin/mysql -h <DB_HOST> -uroot -p -e "create database Joomla"
	sudo /usr/bin/mysql -h <DB_HOST> -uroot -p -e "create user 'user'@'localhost' identified by 'password'"
	sudo /usr/bin/mysql -h <DB_HOST> -uroot -p -e  "GRANT ALL PRIVILEGES ON *.* TO 'user'@'localhost' identified by 'password'"	     

```

4 . Start Apache http server
 ```bash
 sudo /usr/sbin/httpd -D BACKGROUND   (For RHEL 6.8, RHEL 7.1/7.2/7.3 and SLES 12-SP1/12-SP2)
 sudo /usr/sbin/httpd2 -D BACKGROUND  (For SLES 11-SP4 and SLES 12)
 sudo /usr/sbin/apachectl -k start   (For Ubuntu 16.04/16.10)
 ```

### Section 5: Access the Joomla 3.6.4
Access the Joomla install file using a browser
```bash
  	http://<host-ip>:<port>/installation/index.php (for RHEL and SLES)
	http://<host-ip>:<port>/Joomla/installation/index.php (for Ubuntu)
 ```

 _**Note:** If you get an error like "Forbidden You don't have permission to access / on this server" you may need to update the permissions of the parent directory where you installed Apache HTTP Server, for example `sudo chmod o+x <user parent directory>`_

###_[Optional]_ Post installation Setup and Testing 

  * To Manage the site content visit the index file at 

   ```shell
	http://<host-ip>:<port>/index.php (for RHEL and SLES)
	http://<host-ip>:<port>/Joomla/index.php (for Ubuntu)
   ```

###_[Optional]_ Clean up  

  * Remove the ` /<source_root>/` directory to tidy up  

   ```shell
   sudo rm -rf /<source_root>/
   sudo rm /<http_build_location>/Joomla_3.6.4-Stable-Full_Package.zip
   ```
 


###References:  
http://php.net/manual/en/install.unix.apache2.php    
https://docs.joomla.org/J3.x:Installing_Joomla#cite_note-PHPFileHandler-3
