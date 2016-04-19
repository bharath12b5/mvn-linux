<!---PACKAGE:Joomla--->
<!---DISTRO:SLES 12:3.5--->
<!---DISTRO:SLES 11:3.5--->
<!---DISTRO:RHEL 7.1:3.5--->
<!---DISTRO:RHEL 6.6:3.5--->
<!---DISTRO:Ubuntu 16.x:3.5--->

**Joomla** can be built for Linux on z Systems running RHEL 6, RHEL 7 ,SLES 11, SLES12 and Ubuntu 16.04 by following these instructions.  Version 3.5.1 has been successfully built & tested this way.
More information on Joomla is available at https://www.joomla.org and the source code can be downloaded from http://joomlacode.org

_**General Notes:**_

i)  When following the steps below, please use a standard permission user unless otherwise specified.

ii) This recipe uses vi as a standard command line editor but you are welcome to use any alternative line editor of your choice

iii) A directory `/<source_root>/` will be referred to in these instructions.  This is a temporary writeable directory anywhere you'd like to place it.

## Building Joomla

###Obtain pre-built dependencies and create `/<source_root>/` directory.

1. Use the following commands to obtain dependencies

    For RHEL 6.6 and RHEL 7.1
    ```shell
    sudo yum install git wget libjpeg-devel libpng-devel
    ```
    For SLES 11 and SLES 12
    ```shell
    sudo zypper install git wget libjpeg-devel libpng-devel
    ```
    For Ubuntu 16.04

    ```shell
    sudo apt-get update
    sudo apt-get install wget apache2 mysql-server php7.0 php7.0-mysql libapache2-mod-php7.0 libapache2-mod-perl2  php7.0-cli php7.0-common php7.0-curl php7.0-dev
    ```
2. Create the `/<source_root>/` directory as mentioned above

    ```shell
    mkdir /<source_root>/
	  ```

###Build Apache HTTP Server for RHEL/SLES

1. Building Apache HTTP Server 2.5
	
	To build the Apache HTTP Server, refer to the instructions [ here ](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-HTTP-Server)

	
###Dependency Build -  MySQL 5.6.26 (For RHEL/SLES)

1. Install MySQL following the recipe for MySQL.  Run through the steps in [ Building MySQL ](https://github.com/linux-on-ibm-z/docs/wiki/Building-MySQL) recipe up until step 2 under Post installation Setup and Testing.

2. Start the DB if it is not started
	```shell
    sudo <mysql_installation_dir>/bin/mysqld_safe --user=mysql & 
	```
	
3.  Grant access to the mysql user
  ```shell
cd /<mysql_installation_dir>/bin
sudo ./mysql
GRANT ALL PRIVILEGES ON *.* TO 'mysql'@'localhost' IDENTIFIED BY 'mysql'; 
exit
```

###Dependency Build -  php-5.6.2 (For RHEL/SLES)

1. Download the php source code, then extract it.
	
  ```shell
cd /<source_root>/
  wget http://museum.php.net/php5/php-5.6.2.tar.gz
  tar -xzvf php-5.6.2.tar.gz
  cd php-5.6.2
  ``` 

2. Bootstrap to configure the Makefile. Then Make and Install the utility.   

  ```shell
  ./configure --with-apxs2=/<http_build_location>/bin/apxs --with-gd --with-zlib --with-mysql=<mysql_installation_dir>
  make
  sudo make install
  ```
    **Note:**  
i) _`<http_build_location>` is the http server installation prefix location. Default is /usr/local/apache2_  
ii) _`<mysql_installation_dir>` is the mysql installation prefix location. Default is /usr/local/mysql_


3. Setup  php.ini.

  ```shell
  sudo cp php.ini-development /usr/local/lib/php.ini
  ```
	 
4. Update the configuration

    Firstly provide a php configuration:
    ```shell
    sudo cp php.ini-production /usr/local/lib/php.ini
    ```
    Secondly update the Apache configuration to support php:
    ```shell
    sudo vi /usr/local/apache2/conf/httpd.conf
    ```
    Update the config file to have the below section:
    ```shell
    #
    # DirectoryIndex: sets the file that Apache will serve if a directory
    # is requested.
    #
    <IfModule dir_module>
    DirectoryIndex index.php
    </IfModule>
    ```
    _**Note:** We changed `index.html` to `index.php`_  
    
    Update the config file to have the below section:
    ```shell
   #
   # DocumentRoot: The directory out of which you will serve your
   # documents. By default, all requests are taken from this directory, but
   # symbolic links and aliases may be used to point to other locations.
   #

    Define DOCROOT "/usr/local/apache2/htdocs"
    DocumentRoot "${DOCROOT}"

    <Directory "${DOCROOT}"> 
    Options Indexes FollowSymLinks 
    AllowOverride All 
    Require all granted 
    </Directory>
    ```
    _**Note:** We changed `AllowOverride None` to `AllowOverride All`. This will enable the Apache server rewrites._
   
   Update the config file to have the below line change :
   
   ```shell
   LoadModule rewrite_module modules/mod_rewrite.so 
   ```
   _**Note:** Uncomment the line by removing "#" . This will load module for Apache server rewrites._
   
    Also append the following to the end of the config file:
    ```shell
    <FilesMatch \.php$>
    SetHandler application/x-httpd-php    
    </FilesMatch>
    ```
    _**Note:** This section tells Apache how to handle `.php` files, like `index.php` above_
        
    Finally restart the Apache HTTP server to make the configuration take effect:
    ```shell
    sudo /usr/local/apache2/bin/apachectl -k restart
    ```

### Start Apache HTTP Server and MySQL Sever (For Ubuntu 16.04)

1. Start Apache HTTP Server
    ```shell
    sudo /usr/sbin/apachectl  -k start
    
	```

2. Start the DB if it is not started
	```shell
    sudo /usr/bin/mysqld_safe --user=mysql & 
    
	```
	
3.  Grant access to the MySQL user
      ```shell
    /usr/bin/mysql -u root -p
    GRANT ALL PRIVILEGES ON *.* TO 'mysql'@'localhost'; 
    exit
      
      ```

###Product Build -  Joomla 3.4.3

1. Extract Joomla source code 
   
   For RHEL and SLES
  ```shell
  cd /<source_root>/
  wget http://joomlacode.org/gf/download/frsrelease/20086/162539/Joomla_3.4.3-Stable-Full_Package.tar.gz
  sudo cp Joomla_3.4.3-Stable-Full_Package.tar.gz /<http_build_location>/htdocs
  mkdir -p /<http_build_location>/htdocs/Joomla
  cd  /<http_build_location>/htdocs/Joomla
  sudo tar -zxvf Joomla_3.4.3-Stable-Full_Package.tar.gz
  ```
   For Ubuntu 16.04
  ```shell
  cd /var/www/html
  sudo mkdir Joomla
  cd Joomla
  sudo wget https://github.com/joomla/joomla-cms/releases/download/3.5.1/Joomla_3.5.1-Stable-Full_Package.zip
  sudo unzip Joomla_3.5.1-Stable-Full_Package.zip
  ```

2. Move the configuration file to configuration.php 

  ```shell
  cd installation
  sudo mv configuration.php-dist configuration.php
  ```

3. Change the owner of htdocs to daemon

  ```shell
  cd  /<http_build_location>
  sudo chown -R daemon:daemon htdocs
  ```
  
4. Install Joomla

    i) Start the HTTP Server (if not started)
    
    For RHEL/SLES
    ```shell
    /<http_build_location>/bin/apachectl configtest
  	sudo /<http_build_location>/bin/apachectl -k start
    ```
    For Ubuntu 16.04
    ```shell
    sudo /usr/sbin/apachectl  -k start
    ```    

    ii) Start the MySQL Database (if not started)

    For RHEL/SLES
    ```shell
    sudo <mysql_installation_dir>/bin/mysqld_safe --user=mysql &
    ```
    For Ubuntu 16.04
    ```shell
    sudo /usr/bin/mysqld_safe --user=mysql &
    ```

	  iii) Access the Joomla install file using a browser
	
    ```shell
  	http://<host-ip>:<port>/installation/index.php
    ```
     	
    _**Note:** if you get an error like "Forbidden You don't 	have permission to access / on this server" you may need 	to update the permissions of the parent directory where 	you installed Apache HTTP Server, for example `sudo chmod 	o+x <user parent directory>`_

###_[Optional]_ Post installation Setup and Testing. 

1. To Manage the site content visit the index file at 

    ```shell
	 http://<host-ip>:<port>/Joomla/index.php
    ```

###_[Optional]_ Clean up.
1.    Remove the ` /<source_root>/` directory to tidy up.

    ```shell
    sudo rm -rf /<source_root>/
    sudo rm /<http_build_location>/htdocs/Joomla_3.4.3-Stable-Full_Package.tar.gz
    ```
 

	

###References:  
http://php.net/manual/en/install.unix.apache2.php    
https://docs.joomla.org/J3.x:Installing_Joomla#cite_note-PHPFileHandler-3
