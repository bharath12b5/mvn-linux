<!---PACKAGE:WordPress--->
<!---DISTRO:SLES 12:4.7--->
<!---DISTRO:SLES 11:4.7--->
<!---DISTRO:RHEL 7.1:4.7--->
<!---DISTRO:RHEL 6.6:4.7--->
<!---DISTRO:Ubuntu 16.x:4.7--->

# Building WordPress

Below versions of WordPress are available in respective distributions at the time of this recipe creation:

* Ubuntu 16.04 has `4.4.2`
* Ubuntu 16.10 has `4.6.1`

The instructions provided below specify the steps to build WordPress version 4.7 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	 
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##Step 1: Building and Installing WordPress
####1.1) Install dependencies
* RHEL 6.8
  ```sh
  sudo yum install httpd php php-mysql git mysql mysql-server 
  ```

* RHEL 7.1/7.2/7.3
  ```sh
  sudo yum install httpd php php-mysql git mariadb mariadb-server 
  ```

* SLES 11-SP4
  ```sh
  sudo zypper install apache2 apache2-devel tar wget git-core gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel mysql git
  ```

* SLES 12/12-SP1/12-SP2
  ```sh
  sudo zypper install apache2-devel tar wget git-core mariadb gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel
  ```

* Ubuntu 16.04/16.10
  ```sh
  sudo apt-get update
  sudo apt-get install git apache2 mysql-server php php-mysql libapache2-mod-php
  ```
  
####1.2) Build and Install PHP (for SLES 11-SP4 and SLES 12/12-SP1/12-SP2)
```sh
cd /<source_root>/
wget http://www.php.net/distributions/php-5.6.8.tar.gz 
tar xvzf php-5.6.8.tar.gz && cd php-5.6.8
./configure --prefix=/usr/local/php --with-apxs2=/usr/sbin/apxs2 --with-config-file-path=/usr/local/php --with-mysql --with-zlib
make  
sudo make install
```

####1.3) Make changes in the configuration file 
_(You should be root to do the below changes.)_
* RHEL 6.8 and RHEL 7.1/7.2/7.3
  * `/etc/httpd/conf/httpd.conf`
    ```diff
    @@ -116,7 +116,7 @@
     # documents. By default, all requests are taken from this directory, but
     # symbolic links and aliases may be used to point to other locations.
     #
    -DocumentRoot "/var/www/html"
    +DocumentRoot "/var/www/html/WordPress"

     #
     # Relax access to content within /var/www.
    @@ -128,7 +128,7 @@
     </Directory>

     # Further relax access to the default document root:
    -<Directory "/var/www/html">
    +<Directory "/var/www/html/WordPress">
         #
         # Possible values for the Options directive are "None", "All",
         # or any combination of:
    @@ -351,3 +351,7 @@
     #
     # Load config files in the "/etc/httpd/conf.d" directory, if any.
     IncludeOptional conf.d/*.conf
    +ServerName localhost
    +<Directory />
    + DirectoryIndex index.php
    +</Directory>
    ```

* SLES 11-SP4 and SLES 12/12-SP1/12-SP2

  * `/etc/apache2/default-server.conf` 
    ```diff
    @@ -3,12 +3,12 @@
     # deleted here, or overriden elswhere.
     #
    
    -DocumentRoot "/srv/www/htdocs"
    +DocumentRoot "/srv/www/htdocs/WordPress"

     #
     # Configure the DocumentRoot
     #
    -<Directory "/srv/www/htdocs">
    +<Directory "/srv/www/htdocs/WordPress">
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

* Ubuntu 16.04/16.10
  * `/etc/apache2/apache2.conf`
    ```diff
    @@ -219,3 +219,8 @@
     IncludeOptional sites-enabled/*.conf
    
     # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
    +AddType application/x-httpd-php .php
    +ServerName localhost
    +<Directory />
    + DirectoryIndex index.php
    +</Directory>
    ```

  * `/etc/apache2/sites-available/000-default.conf`
    ```diff
	@@ -9,7 +9,7 @@
            #ServerName www.example.com
    
            ServerAdmin webmaster@localhost
    -       DocumentRoot /var/www/html
    +       DocumentRoot /var/www/html/WordPress

            # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
            # error, crit, alert, emerg.
    ```

####1.4) Download and install WordPress
* Download WordPress source code
  ```sh
  cd /var/www/html   #RHEL 6.8, RHEL 7.1/7.2/7.3 and Ubuntu 16.04/16.10
  cd /srv/www/htdocs #SLES 11-SP4 and SLES 12/12-SP1/12-SP2
  sudo git clone https://github.com/WordPress/WordPress.git
  cd WordPress
  sudo git checkout 4.7
  ```

* Rename `wp-config-sample.php` as `wp-config.php` 
  ```sh
  sudo mv wp-config-sample.php wp-config.php 
  ```

* Make the following changes in the `wp-config.php` file ( You may require root permissions to do the changes.) 
  ```diff
  @@ -20,16 +20,16 @@

   // ** MySQL settings - You can get this info from your web host ** //
   /** The name of the database for WordPress */
  -define('DB_NAME', 'database_name_here');
  +define('DB_NAME', 'WORDPRESS');
  
   /** MySQL database username */
  -define('DB_USER', 'username_here');
  +define('DB_USER', 'Wordpress');
  
   /** MySQL database password */
  -define('DB_PASSWORD', 'password_here');
  +define('DB_PASSWORD', 'password');

   /** MySQL hostname */
  -define('DB_HOST', 'localhost');
  +define('DB_HOST', '127.0.0.1');
  
   /** Database Charset to use in creating database tables. */
   define('DB_CHARSET', 'utf8');
  ```

####1.5) Initialize MySQL server
* RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4 and SLES 12/12-SP1/12-SP2
  ```sh
  sudo /usr/bin/mysql_install_db --user=mysql
  ```

* Ubuntu 16.04/16.10
  ```sh
  sudo /usr/sbin/mysqld --initialize --user=mysql --datadir=/var/lib/mysql/data
  ```

####1.6) Create database and grant privileges to 'Wordpress' user
* Modify `/usr/bin/mysqld_safe` **(for SLES 12-SP1/12-SP2)**
  ```diff
  @@ -698,7 +698,7 @@

   if test -z "$pid_file"
   then
  -  pid_file="`hostname`.pid"
  +  pid_file="`echo $HOSTNAME`.pid"
   fi
   # MariaDB wants pid file without datadir
   append_arg_to_args "--pid-file=$pid_file"

  ```

* Create database and grant privileges to 'Wordpress' user
  ```sh
  sudo mkdir -p  /var/log/mysql #(For SLES 11-SP4, SLES 12/12-SP1/12-SP2)
  sudo /usr/bin/mysqld_safe --user=mysql & 
  sudo /usr/bin/mysql  -h <DB_HOST> -uroot -p -e "create database WORDPRESS" && sudo /usr/bin/mysql -h <DB_HOST> -uroot -p -e "create user 'Wordpress'@'localhost' identified by 'password'" 
  sudo /usr/bin/mysql  -h <DB_HOST> -uroot -p -e "grant all privileges on WORDPRESS.* to 'Wordpress'@'localhost' identified by 'password' with GRANT OPTION"
  ```

####1.7) Start httpd server
 ```sh
 sudo /usr/sbin/httpd -D BACKGROUND   #(For RHEL 6.8, RHEL 7.1/7.2/7.3 and SLES 12-SP1/12-SP2)
 sudo /usr/sbin/httpd2 -D BACKGROUND  #(For SLES 11-SP4 , SLES 12)
 sudo service apache2 start           #(For Ubuntu 16.04/16.10)
 ```

_**Note:** After starting Apache httpd Server, direct your Web browser to the WordPress Admin Console at `http://<HOST_IP>:<PORT>`._

## References
https://wordpress.org/
