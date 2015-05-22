# Building Apache Http Web Server

Recipe for Apache Http 2.5.

The following build instructions have been tested with Apache HTTP 2.5 on RHEL7 and SLES12 on IBM z Systems.

### Version
2.5

###Section 1: Install the following Dependencies
* git
* openssl
* gcc
* libtool
* autoconf
* make
* pcre 
* pcre-devel 
* libxml2 
* libxml2-devel
* expat-devel (or libexpat-devel)

RHEL7:
```
yum install -y git \
	openssl \
	gcc \
	libtool \
	autoconf \
	make \
	pcre \
	pcre-devel \
	libxml2 \
	libxml2-devel \
	expat-devel
```

SLES12:
```
zypper install -y git \
	openssl \
	gcc \
	libtool \
	autoconf \
	make \
	pcre \
	pcre-devel \
	libxml2 \
	libxml2-devel \
	libexpat-devel
```


###Section 2. Build and install Httpd
1. (For Rhel) Re-install the ca certificates for openssl framework.
        yum reinstall -y ca-certificates
2. Get the source
        git clone https://github.com/apache/httpd.git
3. Build and Install Apache Portable Runtime library
    * Get the apr source 
            cd httpd/srclib 
            git clone https://github.com/apache/apr.git
    * Configure the build
            cd apr 
           ./buildconf
           ./configure
    * Build the source
            make
    * Install the binaries
            make install
4. Configure the build, without options results in installing the httpd to default location
        cd httpd
        ./buildconf
        ./configure --prefix=<build-location>
5. Build the httpd source
        make
6. Install the binaries
        make install
7. Run the httpd service
        <build-location>/bin/httpd

##References:
https://github.com/apache/httpd	
