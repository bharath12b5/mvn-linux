### Building WordPress

WordPress version 4.3.1 has been tested for Linux on z Systems. The following instructions can be used for RHEL 7.1/6.6 and SLES 12/11.

_**General Notes:**_ 	 
_When following the steps below please use a standard permission user unless otherwise specified._
	

### Section 1: Install dependencies

1.  Apache (with PHP enabled)

    * RHEL7.1/6.6:
	```
		yum install -y httpd php* --skip-broken
	```
	Edit configuration file /etc/apache2/httpd.conf and make the following changes.
	
	Add the following line at the end of the file.
	```
		ServerName localhost
	```
	
    * SLES12/11:
	```
		zypper install -y apache2 apache2-devel tar wget gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel
	```
		
	Edit configuration file /etc/apache2/httpd.conf and make the following changes.
	
	Add the following line at the end of the file.
	```
		ServerName localhost
	```
	Comment or remove the following line
	```
		Include /etc/apache2/sysconfig.d/include.conf
	```
2.  Build and install php using the following steps
     
     ```
		wget http://www.php.net/distributions/php-5.6.8.tar.gz
        tar xvzf php-5.6.8.tar.gz
		cd php-5.6.8
        ./configure --prefix=/usr/local/php --with-apxs2=/usr/sbin/apxs2 --with-config-file-path=/usr/local/php --with-mysql
         make
         make install
	 ```
3.  Enable PHP module 

    Add following lines in httpd.conf (Located at folder /etc/apache2 on SLES12/11 and /etc/httpd/conf on RHEL7.1/6.6)

	RHEL7.1/6.6:
	```
		AddType application/x-httpd-php .php
		<Directory />\n DirectoryIndex index.php \n </Directory>
	```

	SLES12/11:
	```
		AddType application/x-httpd-php .php"
		<Directory />\n DirectoryIndex index.php \n </Directory>
		LoadModule php5_module /usr/lib64/apache2/libphp5.so
	```		
4.  MySQL 

	RHEL7.1/6.6:
	```
		yum install -y mysql   \
                mariadb \
                mariadb-server
	```
	SLES12
	 ```
		zypper install -y mariadb 
	```
	SLES11
	 ```
		zypper install -y mysql 
	```
	  
	
5. Install git
	
	RHEL7.1/6.6:
	```
		yum install -y git
	```

	SLES12/11:
	```
		zypper install -y git
	```

### Section 2: Download WordPress source code and configure

1.  Change to DocumentRoot and download the source in the following location

	RHEL7.1/6.6:
	```
	  cd /var/www/html
	```

	SLES12/11:
	```
		cd /srv/www/htdocs 
	```			
        git clone -b 4.3.1 https://github.com/WordPress/WordPress.git

2.  Create 'WordPress' Database.

                mysql -h <DB_HOST> -uroot -p -e "create database WordPress" ;

3.  Create a new mysql user.

                mysql -h <DB_HOST> -uroot -p -e "create user 'wpuser'@'<ip-address>' identified by 'wpwd'";

    To get ip-address of machine you can use following command.

                hostname -i

4.  Grant all privileges to new user on 'WordPress' database.

                mysql -h <DB_HOST> -uroot -p -e "grant all privileges on WordPress.* to 'wpuser'@'<ip-address>' identified by 'wpwd'";

5.  Copy wp-config-sample.php into wp-config.php.
	
	RHEL7.1/6.6:
	```
		mv /var/www/html/WordPress/wp-config-sample.php      
		/var/www/html/WordPress/wp-config-sample.php/wp-config.php 
	```

	SLES12/11:
	```
		mv /srv/www/htdocs/WordPress/wp-config-sample.php
		/srv/www/htdocs/WordPress/wp-config.php
	```	
        Change Database name to WordPress in wp-config.php.
        Change UserName to WordPress in wp-config.php
	Change Password tp password in wp-config.php
                 

6.  Change DocumentRoot to WordPress folder
	
	RHEL7.1/6.6:
	
	    replace /var/www/html with /var/www/html/WordPress	in /etc/httpd/conf/httpd.conf
	
	SLES12/11:
	    replace srv/www/htdocs with srv/www/htdocs/WordPress in /etc/apache2/default-server.conf 	


7. Start Apache HTTP Server.

      /usr/local/bin/httpd -D FOREGROUND


### References:

     https://github.com/WordPress/WordPress.git/
