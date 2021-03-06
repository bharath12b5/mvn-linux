<!---PACKAGE:Magento--->
<!---DISTRO:SLES 12:2.0--->
<!---DISTRO:SLES 11:2.0--->
<!---DISTRO:RHEL 7.1:2.0--->
<!---DISTRO:RHEL 6.6:2.0--->
<!---DISTRO:Ubuntu 16.x:2.0--->

## Building Magento
The following build instructions have been tested with Magento 2.0.x on RHEL 7.1 & 6.6, SLES 12 & 11 and Ubuntu 16.04 on IBM Linux on z Systems.

_**General Notes:**_  
_i) The solution has been built utilising MySQL as the database layer without any storage or cache storage extensions._  
_ii) When following the steps below please use a standard permission user unless otherwise specified._  
_iii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._  
_iv) For convenience `vi` has been used in the instructions below when editing files, replace with your desired editing program if required._  

## Building / Installing Magento CE
1. Install build dependencies

   RHEL 7.1 & 6.6
    ```shell
    sudo yum install libjpeg-devel libpng-devel curl-devel cronie openssh libicu-devel libxslt-devel.s390x libicu-devel.s390x
    ```
    SLES 12
    ```shell
    sudo zypper install libjpeg8-devel libpng-devel curl-devel cronie openssh libicu-devel pkg-config libxslt-devel
    ```
    SLES 11
    ```shell
    sudo zypper install libjpeg-devel libpng-devel curl-devel cron openssh libmcrypt-devel libicu-devel pkg-config libxslt-devel
    ```
    _**Note:** This recipe assumes that you will be building the MySQL and Apache HTTP servers and will therefore install 
               all of the build dependencies for these._ 

2. MySQL server

   * RHEL 7.1/6.6 and SLES 12/11:

     Please refer to the [MySQL server recipe](https://github.com/linux-on-ibm-z/docs/wiki/Building-MySQL) and
     install the MySQL Server into the default location.

     _ **Note:** Magento requires that the MySQL environment be configured and initialized, which can be achieved by
     following steps **1, 2 and 3** in the optional post install setup and testing section of the recipe._

     Additional information can be found on the MySQL website and community pages below if required.

     [ MySQL 5.6 Reference ](http://dev.mysql.com/doc/refman/5.6/en/index.html)  
     [ MySQL 5.6 Security in MySQL ](http://dev.mysql.com/doc/mysql-security-excerpt/5.6/en/index.html)
	
     Finally ensure that the MySQL server instance is started:
     ```shell
        sudo /usr/local/mysql/bin/mysqld_safe --user=mysql &
     ```

   * Ubuntu 16.04:

     Install dependencies:
     ```shell
        sudo apt-get install mysql-server php7.0-mysql 
     ```
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
      Ensure that the MySQL server instance is started ( start the instance if not started ):
      ```shell
         sudo service mysql status
         sudo service mysql start
      ```  
	
      Secure the installation.
      ```shell
         sudo mysql_secure_installation 
      ```
      **Configure a MySQL database instance:**  

      Test the installation by entering the following command:  
  
      ```shell
         mysql -u root -p
      ```
      Create a database instance named `magento` with user name `magento`: 

      ```shell
         GRANT ALL ON magento.* TO magento@localhost IDENTIFIED BY 'magento';_
      ```

3. Apache HTTP server

   * RHEL 7.1/6.6 and SLES 12/11:

     Please refer to the [Apache HTTP Server recipe](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-HTTP-
     Server) and install the Apache HTTP Server into the default location.
	 
     Whilst Magento can utilise the Apache HTTP as installed from the recipe above, additional information on 
     configuration can be found on the Apache website and community pages below if required.

     [ Apache HTTP Server Reference ](http://httpd.apache.org/docs/2.4/)  
     [ Apache HTTP Server security tips ](http://httpd.apache.org/docs/2.4/misc/security_tips.html)
    
     Finally start the apache2 server:
     ```shell
        <apache2-build-location>/bin/apachectl -k start
     ```

   * Ubuntu 16.04:
 
     Install apache server
     ```shell
	sudo apt-get install apache2
     ```

     Modify apache2.conf file to add domain name 

     Add "ServerName localhost" to `/etc/apache2/apache2.conf` at EOF. 
 
     Enable Apache 2.4 rewrite module  
     ```shell
        sudo a2enmod rewrite
     ```

     Add following lines in Apache 2.4 server's default site configuration file `/etc/apache2/sites-available/000-
     default.conf`,use sudo to open the mentioned file.
     ```shell  
        <Directory "/var/www/html">
              AllowOverride  All
        </Directory>
     ```

     Restart Apache and check status.  
     ```shell
        sudo service apache2 restart
        sudo service apache2 status_
     ```

4. Install libmcrypt

   * RHEL 7.1/6.6 and SLES 12/11:

     First download and extract the correct version of libmcrypt:
     ```shell
        cd /<source_root>/
        wget http://sourceforge.net/projects/mcrypt/files/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz
        tar -xzvf libmcrypt-2.5.8.tar.gz
        cd libmcrypt-2.5.8
     ```
     Then configure, build and install it:
     ```shell
        ./configure
        make
        sudo make install
        export LD_LIBRARY_PATH=/usr/local/lib:/usr/lib:$LD_LIBRARY_PATH
     ```

     _**Note:** The `LD_LIBRARY_PATH` needs to be set correctly, if you have issues later please ensure this is set_

5. Build and configure php

   * RHEL 7.1/6.6 and SLES 12/11:

     _**Note:**  In this section it is assumed that the Apache HTTP and MySQL servers have been installed into their 
                 default locations. If this is not the case then substitute your Apache HTTP server install location to 
                 `/usr/local/apache2` and your MySQL server install location to `/usr/local/mysql` in the steps below._


    1. Download and extract
    
        ```shell
           cd /<source_root>/
           wget http://museum.php.net/php5/php-5.5.9.tar.gz
           tar -xzvf php-5.5.9.tar.gz
           cd php-5.5.9
        ```
    
    2. Generate the configuration

        ```shell
           ./configure --with-apxs2=/usr/local/apache2/bin/apxs --enable-soap --with-libxml-dir=/usr/lib64 --with-gd --
           with-jpeg-dir --with-png-dir --with-pdo_mysql --with-mysql=/usr/local/mysql --with-curl  --with-mcrypt --
           enable-mbstring --enable-intl --enable-zip --with-openssl --with-xsl
        ```
    3. Update the generated Makefile( SLES12 only)

        ```shell
           sudo vi Makefile
        ```
        Change the EXTRA_LIBS lib such that the last -lcrypt becomes -lcrypto. It should look similar to the following 
        after the change
        ```shell
           EXTRA_LIBS = -lcrypt -lresolv -lcrypt -lrt -lmysqlclient_r -lpng -lz -ljpeg -lcurl -lrt -lm -ldl -lnsl -lxml2 -
                         lz -lm -ldl -lcurl -lxml2 -lz -lm -ldl -lxml2 -lz -lm -ldl -lxml2 -lz -lm -ldl -lcrypt -lxml2 -lz 
                         -lm -ldl -lxml2 -lz -lm -ldl -lxml2 -lz -lm -ldl -lcrypto
        ```
    4. Build and install
    
        ```shell
            make
            sudo make install
        ```

     **Update the configuration**:

       Provide a php configuration:
       ```shell
          sudo cp php.ini-production /usr/local/lib/php.ini
       ``` 

       Update Apache configuration to support php:
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

       _**Note:** Change `index.html` to `index.php`_

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


       _**Note:** Change `AllowOverride None` to `AllowOverride All`. This will enable the apache server rewrites._

       Update the config file to have the below line change:
   
       ```shell
         LoadModule rewrite_module modules/mod_rewrite.so 
       ```  

       _**Note:** Uncomment above line will load module for apache server rewrites._

       Also append the following to the end of the config file:  

       ```shell
          <FilesMatch \.php$>
          SetHandler application/x-httpd-php    
          </FilesMatch>
       ```
  
       _**Note:** This section tells apache how to handle `.php` files, like `index.php` above_
   
       Finally restart the Apache HTTP server to make the configuration take effect: 

       ```shell
          sudo /usr/local/apache2/bin/apachectl -k restart
       ```

   * Ubuntu 16.04:

     ```shell
        sudo apt-get install php libapache2-mod-php7.0 php7.0-mysql php7.0-curl php7.0-json php7.0-xsl php7.0-xml php7.0-
        mcrypt php7.0-mbstring php7.0-intl php7.0-zip php7.0-gd
     ```

     Restart apache server:

     ```shell
        sudo service apache2 restart
     ```

6. Install Magento CE

   To install Magento CE refer to the [Magento getting started guide](http://devdocs.magento.com/guides/v2.0/install-
   gde/prereq/zip_install.html#zip-prereq).
    
   _Notes:_  
   _i) If this Magento installation is for testing purposes then it is recommended that you also install the sample data - 
       running through the installation steps for Magento CE will both test the installation and setup Magento CE on your 
       system._  
   ii) While installing Magento on SLES12, if you encounter the error "installation-validate-class-not-found-from-
        basename-stringlength" try the solution below.

  Update the file `vendor/magento/zendframework1/library/Zend/Validate/StringLength.php`  in Magento root with following 
  section:_   
	
   ```shell
      /**
      * Defined by Zend_Validate_Interface
      *
      * Returns true if and only if the string length of $value is at least the min option and
      * no greater than the max option (when the max option is not null).
      *
      * @param  string $value
      * @return boolean
      */

      public function isValid($value)
      {
        if (!is_string($value)) {
            $this->_error(self::INVALID);
            return false;
        }

        $this->_setValue($value);
        if ($this->_encoding !== null) {
            $length = iconv_strlen($value, $this->_encoding);
        } else {
            $length = strlen($value);
        }

        if ($length < $this->_min) {
            $this->_error(self::TOO_SHORT);
        }

        if (null !== $this->_max && $this->_max < $length) {
            $this->_error(self::TOO_LONG);
        }

        if (count($this->_messages)) {
            return false;
        } else {
            return true;
        }
     }
    
   ```
    _**Note:** We changed `$length = iconv_strlen($value);` to `$length = strlen($value);`_

7. **Optionally** Clean up

    Remove (or empty) the ` /<source_root>/` directory to tidy up:
    ```shell
       cd /<source_root>/
       cd ..
       sudo rm -rf /<source_root>/
    ```

### References:  
https://github.com/linux-on-ibm-z/docs/wiki/Building-MySQL  
http://dev.mysql.com/doc/refman/5.6/en/index.html  
http://dev.mysql.com/doc/mysql-security-excerpt/5.6/en/index.html  
https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-HTTP-Server  
http://php.net/manual/en/install.unix.apache2.php  
http://httpd.apache.org/docs/2.4/  
http://httpd.apache.org/docs/2.4/misc/security_tips.html  
http://devdocs.magento.com/guides/v2.0/install-gde/install-quick-ref.html 