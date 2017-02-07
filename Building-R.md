<!---PACKAGE:R--->
<!---DISTRO:SLES 12:3.3.2--->
<!---DISTRO:SLES 11:3.3.2--->
<!---DISTRO:RHEL 7.1:3.3.2--->
<!---DISTRO:RHEL 6.6:3.3.2--->
<!---DISTRO:Ubuntu 16.x:3.3.2--->

# Building R

Below versions of R are available in respective distributions at the time of this recipe creation:

* Ubuntu 16.04 has `3.2.3`
* Ubuntu 16.10 has `3.3.1`

The instructions provided below specify the steps to build R 3.3.2 on IBM z Systems for RHEL 6.8 RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10:

_**General Notes:**_
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

##Step 1: Building and Installing R
####1.1) Install dependencies

  * RHEL 6.8
    ```bash
    sudo yum install wget tar make rpm-build.s390x help2man  git ncurses-devel cairo-devel gcc-c++ libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo gcc-fortran libXt-devel  perl-Text-Unidecode.noarch java-1.8.0-ibm.s390x
    ```

  * RHEL 7.1/7.2/7.3
    * With IBM JDK
      ```bash
      sudo yum install wget tar make rpm-build.s390x help2man zlib-devel xz-devel ncurses-devel cairo-devel gcc-c++ libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel  texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel perl-macros texinfo gcc-gfortran libXt-devel perl-Text-Unidecode.noarch bzip2-devel pcre-devel java-1.7.1-ibm-devel.s390x  java-atk-wrapper.s390x javapackages-tools.noarch
      ```

    * With OpenJDK
      ```bash
      sudo yum install wget tar make rpm-build.s390x help2man zlib-devel xz-devel ncurses-devel cairo-devel gcc-c++ libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo gcc-gfortran libXt-devel java-1.7.0-openjdk-devel.s390x perl-Text-Unidecode.noarch bzip2-devel pcre-devel
      ```

  * SLES 11-SP4
    ```bash
    sudo zypper install wget tar help2man ncurses-devel make cairo-devel gcc-c++ libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel texinfo posix_cc posix_cc-debuginfo posix_cc-debugsource java-1_7_0-ibm git php53-posix man-pages-posix glibc-locale
    sudo zypper install -y gcc45-fortran
    sudo ln -fs /usr/bin/gfortran-4.5 /usr/bin/gfortran
    sudo mkdir $HOME/packages/lib
    sudo cp /usr/lib64/gcc/s390x-suse-linux/4.5/libgfortran* $HOME/packages/lib/ 
    
    ```

  * SLES 12/12-SP1/12-SP2
    * With IBM JDK
      ```bash
      sudo zypper install wget tar rpm-build help2man zlib-devel xz-devel ncurses-devel make cairo-devel gcc-c++ gcc-fortran libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo java-1_7_1-ibm-devel
      ```

    * With OpenJDK
      ```bash
      sudo zypper install wget tar rpm-build help2man zlib-devel xz-devel ncurses-devel make cairo-devel gcc-c++ gcc-fortran libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo java-1.7.0-openjdk-devel.s390x
      ```

  * Ubuntu 16.04/16.10
    * With IBM JDK
      ```bash
      sudo apt-get update
      sudo apt-get install wget tar gcc g++ ratfor gfortran-4.7 gfortran-4.8 libx11-dev make r-base libcurl4-openssl-dev
      ```

      Install IBM Java 8

      Download IBM Java 8 SDK binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per the given link.

    * With OpenJDK
      ```bash
      sudo apt-get update
      sudo apt-get install wget tar gcc g++ ratfor gfortran-4.7 gfortran-4.8 libx11-dev make r-base openjdk-8-jdk libcurl4-openssl-dev
      ```

####1.2) Set environmental variables

  * Set `JAVA_HOME`
    * RHEL
      ```bash
      export JAVA_HOME=/usr/lib/jvm/java
      ```
	  
    * SLES
      ```
      export JAVA_HOME=/usr/lib64/jvm/java
      ```

    * Ubuntu
      * With IBM JDK
        ```bash
        export JAVA_HOME=/<user_install_dir>/ibm/java
        ```
        _**Note:** Where `/<user_install_dir>/` is the location where IBM SKD is installed. Ideally the location is `/opt`._

      * With OpenJDK
        ```bash
        export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
        ```

  * Set other variables (for RHEL 6.8 and SLES 11-SP4)
    ```bash
    export PATH=$HOME/packages/bin:$PATH
    export LD_LIBRARY_PATH=$HOME/packages/lib:$LD_LIBRARY_PATH
    export CFLAGS="-I$HOME/packages/include"
    export LDFLAGS="-L$HOME/packages/lib"
    ```

####1.3) Install other dependencies (for RHEL 6.8 and SLES 11-SP4)

  * Build and install zlib
    ```bash
    cd /<source_root>/
    wget https://sourceforge.net/projects/libpng/files/zlib/1.2.8/zlib-1.2.8.tar.gz/download
    ```
 
    * SLES 11 SP4
       ```bash
       tar xzvf zlib-1.2.8.tar.gz 
       ```

    * RHEL 6.8

        ```bash
        tar xzvf download 
        ```

    ```bash
    cd zlib-1.2.8
    ./configure && make && sudo make install
    ```

  * Build and install bzip
    ```bash
    cd /<source_root>/
    wget http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz
    tar xzvf bzip2-1.0.6.tar.gz
    cd bzip2-1.0.6
    make && sudo make install
    ```

  * Build and install XZ Utils
    ```bash
    cd /<source_root>/
    wget http://tukaani.org/xz/xz-5.2.2.tar.gz
    tar xzvf xz-5.2.2.tar.gz
    cd xz-5.2.2
    ./configure --prefix=$HOME/packages
    make -j3 && sudo make install
    ```

  * Build and install PCRE
    ```bash
    cd /<source_root>/
    wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz
    tar xzvf pcre-8.38.tar.gz
    cd pcre-8.38
    ./configure --enable-utf8 --prefix=$HOME/packages
    make && sudo make install
    ```

  * Build and install OpenSSL
    ```bash
    cd /<source_root>/
    git clone https://github.com/openssl/openssl.git
    cd openssl && git checkout OpenSSL_1_0_2g
    ./config enable-shared && sudo make depend && make && sudo make install
    sudo ln -sf /usr/local/ssl/bin/openssl `which openssl`
    export LD_LIBRARY_PATH=<source_root>/openssl:$LD_LIBRARY_PATH
    sudo ldconfig
    ```

  * Build and install curl
    ```bash
    cd /<source_root>/
    wget --no-check-certificate https://curl.haxx.se/download/curl-7.47.1.tar.gz
    tar xzvf curl-7.47.1.tar.gz
    cd curl-7.47.1
    LDFLAGS=-R/usr/local/ssl/lib ./configure --with-ssl && make && sudo make install
    export LD_LIBRARY_PATH=/<source_root>/curl-7.47.1/lib/.libs:$LD_LIBRARY_PATH
    sudo ldconfig
    ```

####1.4) Download R source code

  ```bash
  cd /<source_root>/
  wget https://cran.r-project.org/src/base/R-3/R-3.3.2.tar.gz
  tar zxvf R-3.3.2.tar.gz
  ```

####1.5) Build and Install R

  ```bash
  cd /<source_root>/R-3.3.2
  ./configure --with-x=no
  make
  sudo make install
  ```

##Step 2: Testing(Optional)

####2.1) Execute test cases

  * For RHEL/SLES
    ```bash
    export LANG="en_US.UTF-8"
    make check
    ```

  * For Ubuntu
    ```bash
    sudo locale-gen "en_US.UTF-8"
    export LANG="en_US.UTF-8"
    make check
    ```

####2.2) Check for version of R-package installed

  ```bash
  echo "sessionInfo()" | R --save
  ```

##References
https://www.r-project.org/
