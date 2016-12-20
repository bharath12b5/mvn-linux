<!---PACKAGE:Apigility--->
<!---DISTRO:SLES 12:1.4.1--->
<!---DISTRO:SLES 11:1.4.1--->
<!---DISTRO:RHEL 7.1:1.4.1--->
<!---DISTRO:RHEL 6.6:1.4.1--->
<!---DISTRO:Ubuntu 16.x:1.4.1--->

# Building Apigility

The instructions provided below specify the steps to build Apigility 1.4.1 on IBM z Systems for RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_
      
* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._


##Step 1: Building and Installing Apigility
####1.1) Install dependencies

* For RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2

    Install Apache Http Server from [here.](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-HTTP-Server)
    
* Other dependencies

	* RHEL 6.8
	  ```
	  sudo yum install -y curl openssl openssl-devel git wget gcc tar ibtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel httpd-devel
	  ```
	
	* RHEL 7.1/7.2/7.3
	  ```
	  sudo yum install -y curl openssl openssl-devel git wget gcc tar libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel  
	  ```
	
	* SLES 11-SP4
	  ```
	  sudo zypper install -y git gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel wget tar awk
	  ```

	* SLES 12/12-SP1/12-SP2
	  ```
	  sudo zypper install -y curl openssl openssl-devel git wget gcc tar libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel
	  ```
	
    * Ubuntu 16.04/16.10
	  ```
	  sudo apt-get update
      sudo apt-get install git apache2 curl openssl make wget tar gcc libssl-dev libxml2 libxml2-dev libxml-parser-perl pkg-config
	  ```

####1.2) Install PHP
  
  * Download PHP
    ```
    cd /<source_root>/
    wget http://www.php.net/distributions/php-5.6.8.tar.gz 
    tar xvzf php-5.6.8.tar.gz && cd php-5.6.8
    ```

  * Configure PHP with Openssl
	
    * RHEL 6.8/7.1/7.2/7.3 and SLES 12/12-SP1/12-SP2
	  ```
      ./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2/bin/apxs --with-config-file-path=/usr/local/php --with-mysql --with-openssl
      ```
	
    * SLES 11-SP4
      ```
      ./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2/bin/apxs --with-config-file-path=/usr/local/php --with-mysql --with-openssl --enable-opcache --with-ldap --with-libdir=lib64
      ```
	
    * Ubuntu 16.04/16.10  
      ```
      ./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php --with-mysql --with-openssl
      ```
	
  * Install PHP
	```
	make
    sudo make install
    ```

  * Set environment path for the PHP
    ```
    export PATH=/usr/local/php/bin:$PATH
    ```

####1.3) Get the source for Apigility
  ```
  cd /<source_root>/
  git clone https://github.com/zfcampus/zf-apigility-skeleton.git 
  cd zf-apigility-skeleton 
  git checkout 1.4.1
  ```

####1.4) Install composer
  ```
  curl -s https://getcomposer.org/installer | php --
  ./composer.phar -n update
  ./composer.phar -n install
  ```

####1.5) Put the `skeleton/app` in development mode
  ```
  ./vendor/bin/zf-development-mode enable
  ```

##Step 2: Start the Apigility application
  ```
  export IP=$(hostname -i)
  cd  /<source_root>/zf-apigility-skeleton
  php -S $IP:8080 -t public public/index.php
  ```
  
## References:

https://github.com/zfcampus/zf-apigility-skeleton
