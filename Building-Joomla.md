<!---PACKAGE:Joomla--->
<!---DISTRO:SLES 12:3.5--->
<!---DISTRO:SLES 11:3.5--->
<!---DISTRO:RHEL 7.1:3.5--->
<!---DISTRO:RHEL 6.6:3.5--->
<!---DISTRO:Ubuntu 16.x:3.5--->


**Joomla** can be built for Linux on z Systems running RHEL 6, RHEL 7, SLES 11, SLES 12 and Ubuntu 16.04 by following these instructions.  Version 3.5.1 has been successfully built & tested this way.
More information on Joomla is available at https://www.joomla.org and the source code can be downloaded from http://joomlacode.org

_**General Notes:**_

i)  When following the steps below, please use a standard permission user unless otherwise specified.

ii) This recipe uses vi as a standard command line editor but you are welcome to use any alternative line editor of your choice

iii) A directory `/<source_root>/` will be referred to in these instructions.  This is a temporary writeable directory anywhere you'd like to place it.

## Building Joomla

### Section 1: Install dependencies

* RHEL 6
```
sudo yum install -y httpd mysql mysql-server wget libjpeg-devel libpng-devel unzip tar gcc gcc-c++ make cmake httpd-devel libxml2 libxml2-devel
```

* RHEL 7
```
sudo yum install -y wget httpd php php-mysql mariadb mariadb-server unzip
```
 
* SLES 11
```
sudo zypper install -y apache2 apache2-devel tar wget gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel mysql libjpeg-devel libpng-devel unzip
```
	 
* SLES 12
```
sudo zypper install -y apache2 apache2-devel tar wget mariadb gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel wget libjpeg-devel libpng-devel unzip
```
* Ubuntu 16.04
```
sudo apt-get update
sudo apt-get install wget apache2 mysql-server php7.0 php7.0-mysql libapache2-mod-php7.0 libapache2-mod-perl2  php7.0-cli php7.0-common php7.0-curl php7.0-dev
```

#####Create the `/<source_root>/` directory as mentioned above
```
mkdir /<source_root>/
```


#####Build and Install PHP (For RHEL 6, SLES 11 and SLES 12)

1 . Download PHP source code
```
cd /<source_root>/
wget http://www.php.net/distributions/php-5.6.8.tar.gz 
tar xvzf php-5.6.8.tar.gz
```

2 . Configure

* RHEL 6
```
cd /<source_root>/php-5.6.8
./configure --prefix=/usr/local/php --with-apxs2=/usr/sbin/apxs --with-config-file-path=/usr/local/php --with-mysql --with-gd --with-zlib 
```

* SLES 11 and SLES 12
```
cd /<source_root>/php-5.6.8
./configure --prefix=/usr/local/php --with-apxs2=/usr/sbin/apxs2 --with-config-file-path=/usr/local/php --with-mysql --with-gd --with-zlib 
```

3 . Build PHP
```
cd /<source_root>/php-5.6.8
make  
sudo make install
```

###Section 2: Make Changes in the configuration files
( You may require root permissions to do the changes.) 

* RHEL 6:

  1 . Edit configuration file `/etc/httpd/conf/httpd.conf` as follows.
       
   * Replace ` /var/www/html` by `/var/www/html/Joomla`

   * Enable the php module by adding the below lines at the end of the configuration file.	
      ```
      AddType application/x-httpd-php .php

     <IfModule dir_module>
           DirectoryIndex index.php
     </IfModule>

    <FilesMatch \.php$>
           SetHandler application/x-httpd-php
    </FilesMatch>

      ```
  
* RHEL 7:

  1 . Edit configuration file `/etc/httpd/conf/httpd.conf` as follows.
       
   * Replace ` /var/www/html` by `/var/www/html/Joomla`

     
* SLES 11 and SLES 12:

  1 . Edit configuration file `/etc/apache2/default-server.conf` as follows.
       
   * Replace ` /srv/www/htdocs` by `/srv/www/htdocs/Joomla`
  
  2 . Edit configuration file `/etc/apache2/httpd.conf` as follows.
	    
   * Comment out the below line. 	 
   
    ```
     Include /etc/apache2/sysconfig.d/include.conf 
    ```
   * Enable the php module by adding the below lines at the end of the configuration file. 

    ```
     AddType application/x-httpd-php .php

     <Directory /> 
     DirectoryIndex index.php 
     </Directory>

     LoadModule php5_module /usr/lib64/apache2/libphp5.so
     ```


### Section 3: Download Joomla source code and configure

1 . Download and install Joomla

RHEL 6, RHEL 7 and Ubuntu 16.04:
 ```
  cd /var/www/html
  sudo mkdir Joomla
  cd Joomla
  sudo wget https://github.com/joomla/joomla-cms/releases/download/3.5.1/Joomla_3.5.1-Stable-Full_Package.zip
  sudo unzip Joomla_3.5.1-Stable-Full_Package.zip
  sudo chmod a+w /var/www/html/Joomla/
```

SLES 11 and SLES 12:
 ```	
 cd /srv/www/htdocs
 sudo mkdir Joomla
 cd Joomla
 sudo wget -O Joomla_3.5.1-Stable-Full_Package.zip  https://github.com/joomla/joomla-cms/releases/download/3.5.1/Joomla_3.5.1-Stable-Full_Package.zip
 sudo unzip Joomla_3.5.1-Stable-Full_Package.zip
 sudo chmod a+w /srv/www/htdocs/Joomla/
```


2 . Rename configuration.php-dist as configuration.php 
 ```
 sudo mv installation/configuration.php-dist installation/configuration.php 
 ```

### Section 4: Start Apache HTTP Server and MySQL Sever

1. Initialize MySQL server  

```
 sudo /usr/bin/mysql_install_db --user=mysql
```

2 . Start MySQL server
 ```
 sudo mkdir -p /var/log/mysql (For SLES 11 and SLES 12)
 sudo /usr/bin/mysqld_safe --user=mysql &  
 ```

3 . Grant privileges
```
 sudo /usr/bin/mysql -u root -p
 GRANT ALL PRIVILEGES ON *.* TO 'mysql'@'localhost'; 
 exit
```

4 . Start Apache http server
 ```
 sudo /usr/sbin/httpd -D BACKGROUND   (For RHEL 6 and RHEL 7)
 sudo /usr/sbin/httpd2 -D BACKGROUND  (For SLES 11 and SLES 12)
 sudo /usr/sbin/apachectl -k start   (For Ubuntu 16.04)
 ```

### Section 5: Access the Joomla 3.5.1
Access the Joomla install file using a browser
```
  	http://<host-ip>:<port>/installation/index.php
 ```

 _**Note:** if you get an error like "Forbidden You don't have permission to access / on this server" you may need to update the permissions of the parent directory where you installed Apache HTTP Server, for example `sudo chmod o+x <user parent directory>`_

###_[Optional]_ Post installation Setup and Testing. 

1. To Manage the site content visit the index file at 

    ```shell
	 http://<host-ip>:<port>/index.php
    ```

###_[Optional]_ Clean up.
1.    Remove the ` /<source_root>/` directory to tidy up.

    ```shell
    sudo rm -rf /<source_root>/
    sudo rm /<http_build_location>/Joomla_3.5.1-Stable-Full_Package.zip
    ```
 


###References:  
http://php.net/manual/en/install.unix.apache2.php    
https://docs.joomla.org/J3.x:Installing_Joomla#cite_note-PHPFileHandler-3