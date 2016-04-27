<!---PACKAGE:WordPress--->
<!---DISTRO:SLES 12:4.5--->
<!---DISTRO:SLES 11:4.5--->
<!---DISTRO:RHEL 7.1:4.5--->
<!---DISTRO:RHEL 6.6:4.5--->

## Building WordPress

WordPress version 4.5 has been tested for Linux on z Systems. The following instructions can be used for RHEL 7.1/6.6 and SLES 12/11.

_**General Notes:**_ 	 

_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

_iii) For convenience vi has been used in the instructions below when editing files, replace with your desired editing program if required._	

### Section 1: Install dependencies

* RHEL6
```
sudo yum install httpd php php-mysql git mysql mysql-server 
```

* RHEL7
```
sudo yum install httpd php php-mysql git mariadb mariadb-server 
```
 
* SLES11
```
sudo zypper install apache2 apache2-devel tar wget git-core gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel mysql git
```
	 
* SLES12
```
sudo zypper install apache2 apache2-devel tar wget git-core mariadb gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel

```
Build and Install PHP (for SLES11 and SLES12)
```
cd <source_root>
wget http://www.php.net/distributions/php-5.6.8.tar.gz 
tar xvzf php-5.6.8.tar.gz && cd php-5.6.8
./configure --prefix=/usr/local/php --with-apxs2=/usr/sbin/apxs2 --with-config-file-path=/usr/local/php --with-mysql 
make  
sudo make install
```


###Section 2: Make changes in the configuration file 
( You should be root to do the below changes.) 

* RHEL6 and RHEL7

  1 . Edit configuration file `/etc/httpd/conf/httpd.conf` as follows.
       

   * Replace  `/var/www/html` by `/var/www/html/WordPress`

   * Add the server name at the end of the configuration file.
     ```
     ServerName localhost
     ```	
   * Enable the php module by adding the below lines at the end of the configuration file.
	
      ```
      <Directory />
          DirectoryIndex index.php 
      </Directory>
      ```
* SLES11 and SLES12

  1 . Replace `/srv/www/htdocs` by `/srv/www/htdocs/WordPress` in `/etc/apache2/default-server.conf` 

  2 . Edit configuration file `/etc/apache2/httpd.conf` as follows.
	    
   * Remove below line in `/etc/apache2/httpd.conf`  	 
   
      ```
       Include /etc/apache2/sysconfig.d/include.conf 
      ```
   * Enable php module by adding the following lines in the  `/etc/apache2/httpd.conf` file
      ```
       AddType application/x-httpd-php .php

       <Directory /> 
             DirectoryIndex index.php 
       </Directory>

       LoadModule php5_module /usr/lib64/apache2/libphp5.so
       ```

### Section 3: Download WordPress source code and configure

1 . Download and install WordPress

RHEL6 and RHEL7
```
cd /var/www/html
sudo git clone https://github.com/WordPress/WordPress.git
cd WordPress
sudo git checkout 4.5
```

SLES11 and SLES12
```	
cd /srv/www/htdocs
sudo git clone https://github.com/WordPress/WordPress.git
cd WordPress
sudo git checkout 4.5
```

2 . Rename `wp-config-sample.php` as `wp-config.php` 
 ```
 sudo mv wp-config-sample.php wp-config.php 
 ```

3 . Make the following changes in the `wp-config.php` file ( You may require root permissions to do the changes.) 
```
     * Change the 'localhost' to '127.0.0.1'
     * Change 'database_name_here' to 'WORDPRESS'
     * Change 'username_here' to 'Wordpress'
     * Change 'password_here' to 'password'
    
```
4 . Initialize MySQL server
```
sudo /usr/bin/mysql_install_db --user=mysql
```

5 . Create database and grant privileges to 'Wordpress' user	
```
sudo mkdir -p  /var/log/mysql(For SLES11 and SLES12)
sudo /usr/bin/mysqld_safe --user=mysql & 
sudo /usr/bin/mysql  -h <DB_HOST> -uroot -p -e "create database WORDPRESS" && sudo /usr/bin/mysql -h <DB_HOST> -uroot -p -e "create user 'Wordpress'@'localhost' identified by 'password'" 
sudo /usr/bin/mysql  -h <DB_HOST> -uroot -p -e "grant all privileges on WORDPRESS.* to 'Wordpress'@'localhost' identified by 'password' with GRANT OPTION"
```

6 . Start httpd server
 ```
 sudo /usr/sbin/httpd -D BACKGROUND   (For RHEL6 and RHEL7)
 sudo /usr/sbin/httpd2 -D BACKGROUND  (For SLES11 and SLES12)
 ```

7 . After starting WordPress, direct your Web browser to the WordPress Admin Console at
 ```
 http://localhost:<port_exposed>
 ```

#### References
https://github.com/WordPress/WordPress.git/