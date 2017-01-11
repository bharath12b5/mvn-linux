## Building Zabbix server

Below versions of Zabbix server are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04/16.10 has `2.4.7`

The instructions provided below specify the steps to build Zabbix server version 3.2.1 on the IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12/12-SP1/12-SP2, Ubuntu 16.04/16.10.

_**General Notes:**_                  

* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##Step 1: Install dependencies
                                


 * Ubuntu 16.04/16.10 
  ```shell
    sudo apt-get update 
    sudo apt-get -y install wget tar curl vim gcc make snmp snmptrapd ceph libmysqld-dev libmysqlclient-dev libxml2-dev libsnmp-dev libcurl3 libcurl4-openssl-dev git apache2 php php-mysql libapache2-mod-php mysql-server php7.0-xml php7.0-gd php-bcmath php-mbstring 
  ``` 
      
 * RHEL 7.1/7.2/7.3
  ```shell
    sudo yum update
    sudo yum install -y wget tar curl vim gcc make net-snmp.s390x  net-snmp-devel.s390x mariadb mariadb-server.s390x mariadb-devel.s390x php-mysql.s390x mariadb-libs.s390x git httpd.s390x php php-mysql libcurl-devel.s390x libxml2-devel.s390x php-xml.s390x php-gd.s390x php-bcmath php-mbstring 
  ```
 * SLES 12
 ```shell
    sudo zypper update
    sudo zypper install -y wget tar curl vim gcc make net-snmp net-snmp-devel mariadb libmysqld-devel  net-tools git apache2 apache2-devel apache2-mod_php5 php5 php5-mysql php5-xmlreader php5-xmlwriter php5-gd php5-bcmath php5-mbstring php5-ctype php5-sockets php5-gettext libcurl-devel libxml2  libxml2-devel 
 ```

 * SLES 12-SP1/12-SP2
 ```shell
    sudo zypper update
    sudo zypper install -y wget tar curl vim gcc make net-snmp net-snmp-devel net-tools mariadb libmysqld-devel git apache2 apache2-devel libcurl-devel libxml2 libxml2-devel libtool pcre pcre-devel git-core autoconf libexpat-devel freetype freetype-devel  libjpeg-devel libpng-devel freetype2 freetype2-devel
 ```

 * ####Other dependencies    
         
  For SLES 12-SP1/12-SP2, install following additional dependency:   

    * PHP-5.6.8 

       ```shell 
          cd <source_root>   
          wget http://www.php.net/distributions/php-5.6.8.tar.gz  
          tar xvzf php-5.6.8.tar.gz && cd php-5.6.8 
          sudo ./configure --prefix=/usr/local/php --with-apxs2=/usr/sbin/apxs2 --with-config-file-path=/usr/local/php --with-mysql --with-gd --with-zlib  --with-gettext --enable-bcmath --enable-mbstring --enable-sockets  --with-jpeg-dir --with-png-dir --with-jpeg-dir=/usr/include/jpeglib.h --enable-gd-native-ttf --enable-ctype  --with-mysqli --with-freetype-dir=/usr/lib 
          make   
          sudo make install 
          sudo cp /<source_root>/php-5.6.8/php.ini-development /usr/local/php/php.ini   
       ```

##Step 2: Enable PHP support by modifying Apache configuration file

* Ubuntu 16.04/16.10

	  *  Add the server name at the end of configuration file `/etc/apache2/apache2.conf` and enable the php module as shown below 
   
     ```diff  	 
           # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
     +    ServerName localhost
     +    AddType application/x-httpd-php .php
     +    <Directory />
     +    DirectoryIndex index.php 
     +    </Directory>
     ```
* RHEL 7.1/7.2/7.3
 
	  *  Add the server name at the end of configuration file `/etc/httpd/conf/httpd.conf`  and enable the php module as shown below         

     ```diff
           IncludeOptional conf.d/*.conf
     +    ServerName localhost
     +    <Directory />
     +    DirectoryIndex index.php 
     +    </Directory>  
     ```
                 
	
* SLES 12

     * Edit configuration file `/etc/apache2/httpd.conf` as follows
  
	   * Remove below line 
 
	   ```diff
	   -    Include /etc/apache2/sysconfig.d/include.conf
	   ``` 
 

	   * Enable the php module by adding the below lines at the end of the configuration file
	    ```diff
             #     /etc/sysconfig/apache2 -- this will make system updates
             #     easier :)
	    +    AddType application/x-httpd-ph .php
	    +    <Directory /> 
	    +    DirectoryIndex index.php index.html
	    +    </Directory>
	
	    ```

         
     * Include below line in `/etc/apache2/sysconfig.d/loadmodule.conf`
 
	    ```diff
	    +    LoadModule php5_module /usr/lib64/apache2/mod_php5.so
	    ```
     * Enable php module by the command 
    
           ```
           sudo a2enmod php
           ```

* SLES 12-SP1/12-SP2
   
	  *  Add the server name at the end of `/etc/apache2/httpd.conf` configuration file and enable the php module as shown below 

	 ```diff
           #     /etc/sysconfig/apache2 -- this will make system updates
           #     easier :)
	 +    AddType application/x-httpd-php .php
	 +    <Directory /> 
	 +    DirectoryIndex index.php 
	 +    </Directory>
	 +    LoadModule php5_module /usr/lib64/apache2/libphp5.so

	 ```

##Step 3: Download and install Zabbix server

```shell
   cd /<source_root>/
   wget http://sourceforge.net/projects/zabbix/files/ZABBIX%20Latest%20Stable/3.2.1/zabbix-3.2.1.tar.gz/download
   tar -xvf download
   cd /<source_root>/zabbix-3.2.1
   ./configure --enable-server --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2
   make
   sudo make install
```

* Create a 'zabbix' user required to start Zabbix server daemon

  ```shell
   sudo groupadd zabbix
   sudo useradd -g zabbix zabbix
  ```
*  Installing Zabbix web interface
      
       * RHEL 7.1/7.2/7.3, Ubuntu 16.04/16.10
       ```shell
       cd /<source_root>/zabbix-3.2.1/frontends/php/
       sudo mkdir -p /var/www/html/zabbix
       sudo cp -rf * /var/www/html/zabbix/
       cd /var/www/html/zabbix
       sudo chown -R zabbix:zabbix conf 
       ```
                
       * SLES 12/12-SP1/12-SP2
       ```shell
       cd /<source_root>/zabbix-3.2.1/frontends/php/
       sudo mkdir -p /srv/www/htdocs/zabbix  
       sudo cp -rf * /srv/www/htdocs/zabbix           
       cd /srv/www/htdocs/zabbix
       sudo chown -R zabbix:zabbix conf 
       
       ``` 

    

##Step 4: Prerequisites to  start Zabbix server


*  Start httpd server 
  
    ```shell
   sudo service apache2 start          (for Ubuntu 16.04/16.10) 
   sudo /usr/sbin/httpd -D BACKGROUND  (for RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2)
   sudo /usr/sbin/httpd2 -D BACKGROUND (for SLES 12)
    ```
* Start MySQL service  

   _**Note:**_ For reference, click [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-MySQL) 

* Create database and grant privileges to zabbix user
                 
   _**Note:**_ First two commands will prompt for password. Press `Enter` if the password for root user is not set.  
      
  ```shell   
   sudo /usr/bin/mysql -uroot -p -e "create database zabbix" && sudo /usr/bin/mysql -uroot -p -e "create user 'zabbix'@'localhost'"
   sudo /usr/bin/mysql -uroot -p -e "grant all privileges on zabbix.* to 'zabbix'@'localhost'"
   cd /<source_root>/zabbix-3.2.1/database/mysql
   mysql -uzabbix zabbix < schema.sql
   mysql -uzabbix zabbix < images.sql
   mysql -uzabbix zabbix < data.sql
  ```


##Step 5: Change php.ini file

* Edit php parameters values in `php.ini` file as shown below


  Common location of `php.ini`
   * `/etc/php/7.0/apache2/php.ini` ( Ubuntu 16.04/16.10)
   * `/etc/php.ini` (RHEL 7.1/7.2/7.3)
   * `/etc/php5/apache2/php.ini` (SLES 12)
   * `/usr/local/php/php.ini` (SLES 12-SP1/12-SP2)

   _**Note:**_ Update appropriate time zone by modifying below line. 
  
   ```diff
         ; http://php.net/date.timezone
   -    ;date.timezone =
   +    date.timezone = 'Asia/Kolkata'
         ; http://php.net/date.default-latitude
    ```  

   
    ```diff
         ; Note: This directive is hardcoded to 0 for the CLI SAPI
   -    max_execution_time = 30
   +    max_execution_time = 300
         ; Maximum amount of time each script may spend parsing request data. It's a good
   ```

   ```diff
         ; http://php.net/max-input-time
   -    max_input_time = 60
   +    max_input_time = 300
         ; Maximum input variable nesting level
   ```

   ```diff
         ; http://php.net/post-max-size
   -    post_max_size = 8M
   +    post_max_size = 16M
         ; Automatically add files before PHP document.
   ```

   _**Note:**_ Change below line for SLES12-SP1/12-SP2     
   ```diff
         ; http://php.net/always-populate-raw-post-data
   -    ;always_populate_raw_post_data = -1
   +    always_populate_raw_post_data = -1
   ```


##Step 6: Start Zabbix server                
*  Start Zabbix server and restart http service
                
  ```shell
     zabbix_server
     sudo service apache2 restart  (for Ubuntu 16.04/16.10)
     sudo httpd -k restart         (for RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2)
     sudo httpd2 -k restart        (for SLES 12)
  ```
                

*  Verify the installed Zabbix server version with the following command:
  ```shell
     zabbix_server -V
  ```

*  After starting Zabbix server, direct your Web browser to the Zabbix Console by using following URL:
  ```shell
     http://<HOST_IP>:<PORT>/zabbix     
  ```
*  Follow below instruction to configure Zabbix

  
  *  Change `Database host` from `localhost` to `127.0.0.1` if you get the error "Error connecting to database: No such file or directory". 
  *  Download configuration file manually if you get the error "Cannot create the configuration file" and save it in the path specified in Zabbix console.

  *  Login to Zabbix using below default credentials.
  
    ```shell
      User     : Admin
      Password : zabbix
    ```
               

#### Reference:
https://www.zabbix.com/documentation/3.2/manual/installation/

