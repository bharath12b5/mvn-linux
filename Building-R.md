<!---PACKAGE:R--->
<!---DISTRO:SLES 12:3.3.1--->
<!---DISTRO:SLES 11:3.3.1--->
<!---DISTRO:RHEL 7.1:3.3.1--->
<!---DISTRO:RHEL 6.6:3.3.1--->
<!---DISTRO:Ubuntu 16.x:3.3.1--->

## Building R 3.3.1

Below versions of R are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `3.2.3`

The instructions provided below specify the steps to build R 3.3.1 on Linux on the IBM z Systems for RHEL 6.7/7.1/7.2, SLES 11-SP3, SLES 12/12-SP1 and Ubuntu 16.04.

**General Notes:**	 
i)  _When following the steps below please use a standard permission user unless otherwise specified._


1. Installing Build Dependencies

      a) RHEL 7.1/7.2

     * With IBM JDK 
     ```
     sudo yum install -y wget tar make rpm-build.s390x help2man zlib-devel xz-devel ncurses-devel cairo-devel gcc-c++ libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel  texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel perl-macros texinfo gcc-gfortran libXt-devel perl-Text-Unidecode.noarch bzip2-devel pcre-devel java-1.7.1-ibm-devel.s390x  java-atk-wrapper.s390x javapackages-tools.noarch
     ```

     * With OpenJDK  
     ```
     sudo yum install -y wget tar make rpm-build.s390x help2man zlib-devel xz-devel ncurses-devel cairo-devel gcc-c++ libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo gcc-gfortran libXt-devel java-1.7.0-openjdk-devel.s390x perl-Text-Unidecode.noarch bzip2-devel pcre-devel
     ```

  b) SLES 12/12-SP1

     * With IBM JDK 
 
     ```
     sudo zypper install -y wget tar rpm-build help2man zlib-devel xz-devel ncurses-devel make cairo-devel gcc-c++ gcc-fortran libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo java-1_7_1-ibm-devel
     ```

     * With OpenJDK  
     ```
     sudo zypper install -y wget tar rpm-build help2man zlib-devel xz-devel ncurses-devel make cairo-devel gcc-c++ gcc-fortran libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo java-1.7.0-openjdk-devel.s390x
     ```
 
  c) RHEL 6.7
   ```
   sudo yum install -y wget tar make rpm-build.s390x help2man  git ncurses-devel cairo-devel gcc-c++ libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo gcc-gfortran libXt-devel  perl-Text-Unidecode.noarch java-1.8.0-ibm.s390x
   ```
  
  d) SLES 11-SP3
   ```
   sudo zypper install  wget tar help2man ncurses-devel make cairo-devel gcc-c++   libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes   xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel texinfo posix_cc posix_cc-debuginfo posix_cc-debugsource php5-posix php5-posix java-1_7_0-ibm gcc-fortran git php53-posix man-pages-posix
   ```
  e) Ubuntu 16.04

   * With IBM JDK
   
	  Install IBM Java 8
	
	  Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link
	
	  ```
	  sudo apt-get update
	  sudo apt-get install -y wget tar gcc g++ ratfor  gfortran-4.7  gfortran-4.8 libx11-dev make r-base libcurl4-openssl-dev
	  ```
	
	* With OpenJDK
	  ```
	  sudo apt-get update
	  sudo apt-get install wget tar gcc g++ ratfor  gfortran-4.7  gfortran-4.8 libx11-dev make r-base openjdk-8-jdk libcurl4-openssl-dev
	  ```

      
2. Set environmental variables 
  
   RHEL 7.1/7.2
   ```
   export JAVA_HOME=/usr/lib/jvm/java
   ```
   SLES 12/12-SP1 
   ```
   export JAVA_HOME=/usr/lib64/jvm/java
    ```
   RHEL 6.7
   ```
   export HOME=<source_root>
   export JAVA_HOME=/usr/lib/jvm/java
   export PATH=$HOME/packages/bin:$PATH
   export LD_LIBRARY_PATH=$HOME/packages/lib:$LD_LIBRARY_PATH
   export CFLAGS="-I$HOME/packages/include"
   export LDFLAGS="-L$HOME/packages/lib"
    ```
   SLES 11-SP3
   ```
   export HOME=<source_root>
   export JAVA_HOME=/usr/lib64/jvm/java
   export PATH=$HOME/packages/bin:/sbin:$PATH
   export LD_LIBRARY_PATH=$HOME/packages/lib:$LD_LIBRARY_PATH
   export CFLAGS="-I$HOME/packages/include"
   export LDFLAGS="-L$HOME/packages/lib"
    ```
   Ubuntu 16.04     
   
   * With IBM JDK
     ```
     export JAVA_HOME=/<source_root>/ibm/java-s390x-80
     ```
   
   * With OpenJDK
     ```
     export JAVA_HOME=/usr/lib/jvm/java
     ```
   
   _**Note:** Where `<source_root>` is the directory defined at the top of this document - if you wish to clean up the `<source_root>` at the end of the install you may want to place ibm-java-s390x-80 in a different location, if so just update the JAVA_HOME_

      
3. Build and install perl-Text-Unidecode (for SLES 12/12-SP1) 
   
   Download the latest version of perl-Text-Unidecode 
    ```
    wget http://download.opensuse.org/repositories/devel:/languages:/R:/patched/SLE_12/src/perl-Text-Unidecode-*.src.rpm   (* indicates the latest version number) 
    sudo zypper refresh
    rpmbuild --rebuild perl-Text-Unidecode-*.src.rpm
    cp $HOME/rpmbuild/RPMS/noarch/perl-Text-Unidecode-*.noarch.rpm .
    sudo zypper refresh
    sudo zypper install -y perl-Text-Unidecode-*.noarch.rpm
         
    ```
4. Installing other dependencies (for RHEL 6.7 and SLES 11-SP3)

	1. Build and install Zlib 

	  ```
	  cd <source_root>   
	  wget http://zlib.net/zlib-1.2.8.tar.gz
	  tar xzvf zlib-1.2.8.tar.gz
	  cd zlib-1.2.8
	  ./configure && make && sudo make install         
	  ```

	2. Build and install bzip

	  ```
	  cd <source_root>  
 	  wget http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz
      tar xzvf bzip2-1.0.6.tar.gz
	  cd bzip2-1.0.6
	  make && sudo make install  
	  ```

    3. Build and install xz

      ```
	  cd <source_root>  
	  wget http://tukaani.org/xz/xz-5.2.2.tar.gz
	  tar xzvf xz-5.2.2.tar.gz
	  cd xz-5.2.2
	  ./configure --prefix=$HOME/packages
	  make -j3 && sudo make install
      ```
	  
	4. Build and install pcre

      ```
	  cd <source_root>  
	  wget  ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz
	  tar xzvf pcre-8.38.tar.gz
	  cd pcre-8.38
	  ./configure --enable-utf8 --prefix=$HOME/packages
	  make && sudo make install      
      ```
	  
	5. Build and install Openssl

      ```
	  cd <source_root>  
	  git clone https://github.com/openssl/openssl.git && cd openssl && git checkout OpenSSL_1_0_2g && ./config enable-shared && sudo make depend && make && sudo make install
	  sudo ln -sf /usr/local/ssl/bin/openssl `which openssl`
	  export LD_LIBRARY_PATH=<source_root>/openssl:$LD_LIBRARY_PATH
	  sudo ldconfig  
      ```
	  
	6. Build and install curl
  
      ```
	  cd <source_root>  
	  wget --no-check-certificate https://curl.haxx.se/download/curl-7.47.1.tar.gz
	  tar xzvf curl-7.47.1.tar.gz
	  cd curl-7.47.1
	  ./configure --with-ssl && make && sudo make install
	  export LD_LIBRARY_PATH=<source_root>/curl-7.47.1/lib/.libs:$LD_LIBRARY_PATH 
	  sudo ldconfig    
      ```
	  
5. Download R 3.3.1 source code
    
    ```
	cd <source_root>
    wget https://cran.r-project.org/src/base/R-3/R-3.3.1.tar.gz
    tar zxvf R-3.3.1.tar.gz
    ```
6. Build and Install  R 
   
   ```
   cd R-3.3.1
   ./configure --with-x=no
   make
   sudo make install
   ```
7. Execute test cases 

   For RHEL/SLES
   ```
    export LANG="en_US.UTF-8"
    make check     
    ```
   For Ubuntu 16.04
   ```
   sudo locale-gen "en_US.UTF-8"
   export LANG="en_US.UTF-8"
   make check   
   ```

  
8. Check for version of R-package installed
  
   ```
    echo "sessionInfo()" | R --save
   ```
  
####References
https://www.r-project.org/
