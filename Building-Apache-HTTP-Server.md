<!---PACKAGE:Apache HTTP--->
<!---DISTRO:SLES 12:2.4.x--->
<!---DISTRO:SLES 11:2.4.x--->
<!---DISTRO:RHEL 7.1:2.4.x--->
<!---DISTRO:RHEL 6.6:2.4.x--->
<!---DISTRO:Ubuntu 16.x:2.4.x--->

###Building Apache Http Web Server

Below versions of Apache Http Web Server are available in respective distributions at the time of this recipe creation:

*    RHEL 7.2 has `2.4.6`
*    RHEL 6.6 has `2.2.15`
*    SLES 12 has `2.4.10`
*    SLES 11.3 has `2.2.12`
*    Ubuntu 16.04 has `2.4.18`

The instructions provided below specify the steps to build Apache Http Web Server version 2.4.23 on Linux on the IBM z Systems for RHEL 6.6/7.2 and SLES 11/12 and Ubuntu.

**General Notes:**

i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

iii) Where the instructions refer to 'vi' you may, of course, use an editor of your choice

#Building Apache HTTP Web Server

1. Install build dependencies

	For RHEL 7.2 & 6.6

		sudo yum install git openssl openssl-devel gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel expat-devel which wget tar

	For SLES 12 

		sudo zypper install git openssl openssl-devel gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel wget tar
	
	For SLES 11
	
	    sudo zypper install git gcc libtool autoconf make pcre pcre-devel libxml2 libxml2-devel libexpat-devel wget tar
	
	For Ubuntu 16.04
	   
		sudo apt-get install git python openssl gcc autoconf make libtool-bin libpcre3-dev libxml2  libexpat1 libexpat1-dev wget tar 

2. Build Openssl 1.0.2 (SLES 11 Only)
    
		cd /<source_root>/
		wget ftp https://www.openssl.org/source/old/1.0.2/openssl-1.0.2i.tar.gz
		tar zxf openssl-1.0.2i.tar.gz
		cd openssl-1.0.2i
		./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib shared zlib-dynamic
		make
		sudo make install		
		export PATH=/usr/local/ssl/bin:$PATH

3. Build Libtool (SLES 11 & RHEL 6.6 Only)

	SLES 11 and RHEL 6.6 has an older version of libtool available. Apache HTTP Server needs a later version of libtool.

		cd /<source_root>/
		wget http://ftpmirror.gnu.org/libtool/libtool-2.4.6.tar.gz
		tar -xvf libtool-2.4.6.tar.gz
		cd libtool-2.4.6
		./configure
		make
		sudo make install

		
4. Extract Apache HTTP source code (and supporting packages)

		cd /<source_root>/
		git clone https://github.com/apache/httpd.git 
		cd httpd
		git checkout 2.4.23
		cd srclib
		git clone https://github.com/apache/apr.git

5. Build Apache Portable Runtime

		cd apr
		./buildconf 
		./configure
		make
		sudo make install

6. Build and Install Apache HTTP Server

		cd /<source_root>/httpd
		./buildconf
		./configure --prefix=<build-location>
		make
		sudo make install

    _**Note:** Skipping the `--prefix` results in Apache httpd being installed in the default location._

	
7.  Verification

    _**Note:** All the following commands may require `sudo` depending on the `<build-location>` specified_

    Update your configuration as necessary

		vi <build-location>/conf/httpd.conf

    Verify the configuration and start the server

		<build-location>/bin/apachectl configtest
		<build-location>/bin/apachectl -k start
    
	_**Note:** If `<build-location>` is the prefix you specified, ensure that in <build-location>/conf/httpd.conf file,_
	_values for `User` and `Group` fields are same as that of `<build-location>`. Else, it will display forbidden(403) error in the browser._  
    
	Test the webserver

		<build-location>/bin/apachectl -k stop

    _**Note:** `<build-location>` is the prefix you specified on step 5 - if you didn't specify a prefix it should be installed to `/usr/local/apache2`_

#References:
https://github.com/apache/httpd