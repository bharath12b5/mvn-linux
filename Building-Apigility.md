# Building Apigility

Apigility version 1.3.1 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1/6.6 and SLES 12/11.

_**General Notes:**_ 	 
_When following the steps below please use a standard permission user unless otherwise specified._


### Section 1: Install the following dependencies

* Install Apache Http Server from [here.](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-HTTP-Server)
    
* Other dependencies

	RHEL7.1:
	```
	yum install -y curl openssl openssl-devel git wget gcc tar libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel  

	```
	RHEL6.6:
	```
	yum install -y curl openssl openssl-devel git wget gcc tar ibtool autoconf 		make pcre pcre-devel libxml2 libxml2-devel libexpat-devel  httpd-devel
	```

	SLES12:
	```
	zypper install -y curl openssl openssl-devel git wget gcc tar libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel

	```
	
	SLES11:
	```
	zypper install -y curl openssl openssl-devel git wget gcc tar libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libxml2-devel pkg-config apache2 apache2-devel

	```

#### Section 2: Build and install
1. Download, configure and install PHP with openssl
 
	Steps for RHEL7.1 and SLES12
	```
			wget http://www.php.net/distributions/php-5.6.8.tar.gz 
			tar xvzf php-5.6.8.tar.gz && cd php-5.6.8
			./configure --prefix=/usr/local/php --with-apxs2=/usr/local/bin/apxs --with-config-file-path=/usr/local/php --with-mysql --with-openssl
			make
			make install
	```
	
	Steps for RHEL6.6
	```
			wget http://www.php.net/distributions/php-5.6.8.tar.gz 
			tar xvzf php-5.6.8.tar.gz && cd php-5.6.8
			./configure --prefix=/usr/local/php --with-apxs2=/usr/sbin/apxs --with-config-file-path=/usr/local/php --with-mysql --with-openssl
			make
			make install
	```
	
	Steps for SLES11
	```
			wget http://www.php.net/distributions/php-5.6.8.tar.gz 
			tar xvzf php-5.6.8.tar.gz && cd php-5.6.8
			./configure --prefix=/usr/local/php --with-apxs2=/usr/sbin/apxs2 	--with-config-file-path=/usr/local/php --with-mysql --with-openssl
			make 
			make install
	```
2. Set environment path for the PHP

		export PATH=/usr/local/php/bin:$PATH

3. Get the source for Apigility

        cd /
        git clone https://github.com/zfcampus/zf-apigility-skeleton.git 
        cd /zf-apigility-skeleton 
        git checkout 1.3.1

4. Install composer

        curl -s https://getcomposer.org/installer | php --
        ./composer.phar -n update
        ./composer.phar -n install

5. Put the skeleton/app in development mode

        php public/index.php development enable


6. Start the Apigility application

        export IP=$(hostname -i)
        cd  /zf-apigility-skeleton
        php -S $IP:8080 -t public public/index.php

## References:

https://github.com/zfcampus/zf-apigility-skeleton

