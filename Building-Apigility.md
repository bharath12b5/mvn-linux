<!---PACKAGE:Apigility--->
<!---DISTRO:SLES 12:1.4.0--->
<!---DISTRO:SLES 11:1.4.0--->
<!---DISTRO:RHEL 7.1:1.4.0--->
<!---DISTRO:RHEL 6.6:1.4.0--->
<!---DISTRO:Ubuntu 16.x:1.4.0--->

# Building Apigility

The instructions provided below specify the steps to build Apigility 1.4.0 on Linux on the IBM z Systems for RHEL 6/7, SLES 11/12 and Ubuntu 16.04.

##### General Notes:
      
i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it.


### Section 1: Install the following dependencies

* For RHEL 6.6/7.1 and SLES 11/12 

    Install Apache Http Server from [here.](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-HTTP-Server)
    
* Other dependencies

	RHEL7.1:
	```
	sudo yum install -y curl openssl openssl-devel git wget gcc tar libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel  
	```
	RHEL6.6:
	```
	sudo yum install -y curl openssl openssl-devel git wget gcc tar ibtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel  httpd-devel
	```

	SLES12:
	```
	sudo zypper install -y curl openssl openssl-devel git wget gcc tar libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel
	```
	
	SLES11:
	```
	sudo zypper install -y curl openssl openssl-devel git wget gcc tar libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libxml2-devel pkg-config apache2 apache2-devel
	```

    Ubuntu16.04:
	```
	sudo apt-get update
sudo apt-get install git apache2 curl openssl make wget tar gcc libssl-dev libxml2 libxml2-dev libxml-parser-perl pkg-config
	```

### Section 2: Build and Install
1. Download, configure and install PHP with Openssl
  
	Download PHP
	
	```
	cd /<source_root>/
	wget http://www.php.net/distributions/php-5.6.8.tar.gz 
	tar xvzf php-5.6.8.tar.gz && cd php-5.6.8
	```
	
	Configure PHP with Openssl
	
    * For RHEL 6.6/7.1 and SLES 12
	
	```
	./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2/bin/apxs --with-config-file-path=/usr/local/php --with-mysql --with-openssl
	```
	
	* For SLES 11
	
	```
	 ./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2/bin/apxs --with-config-file-path=/usr/local/php --with-mysql --with-openssl --enable-opcache --with-ldap --with-libdir=lib64
	```
	
    * For Ubuntu 16.04
        
    ```
    ./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php --with-mysql --with-openssl
    ```
	
	Install PHP
	```
	make
    sudo make install
    ```

2. Set environment path for the PHP

		export PATH=/usr/local/php/bin:$PATH

3. Get the source for Apigility

        cd /<source_root>/
        git clone https://github.com/zfcampus/zf-apigility-skeleton.git 
        cd zf-apigility-skeleton 
        git checkout 1.4.0

4. Install composer

        curl -s https://getcomposer.org/installer | php --
        ./composer.phar -n update
        ./composer.phar -n install

5. Put the skeleton/app in development mode

        ./vendor/bin/zf-development-mode enable


6. Start the Apigility application

        export IP=$(hostname -i)
        cd  /<source_root>/zf-apigility-skeleton
        php -S $IP:8080 -t public public/index.php

## References:

https://github.com/zfcampus/zf-apigility-skeleton
