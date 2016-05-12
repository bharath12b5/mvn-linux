
## Building R 3.2.5

These instructions describe how to build R(https://www.r-project.org/) for IBM Linux on z system. Version 3.2.5 for RHEL6, RHEL7.1, SLES 12 and Ubuntu 16.04 has been successfully built this way.

**General Notes:**	 
i)  _When following the steps below please use a standard permission user unless otherwise specified._


#### Building R 

1. Installing Build Dependencies
      
   SLES 12     
   ```
   sudo zypper install -y wget tar rpm-build help2man zlib-devel xz-devel ncurses-devel make cairo-devel gcc-c++ gcc-fortran libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo java-1.7.0-openjdk-devel.s390x
      
    ```

   RHEL 6 & RHEL 7.1 
   ```
  sudo yum install -y wget tar make rpm-build.s390x help2man zlib-devel xz-devel ncurses-devel cairo-devel gcc-c++ libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo gcc-gfortran libXt-devel java-1.7.0-openjdk-devel.s390x perl-Text-Unidecode.noarch 
   ```

   Ubuntu 16.04 
   ```
  sudo apt-get update
  sudo apt-get install wget tar gcc g++ ratfor  gfortran-4.7  gfortran-4.8 libx11-dev make r-base openjdk-8-jdk
   ```

      
2. Build and install perl-Text-Unidecode (for SLES12) 
   
   Download the latest version of perl-Text-Unidecode 
    ```
    wget http://download.opensuse.org/repositories/devel:/languages:/R:/patched/SLE_12/src/perl-Text-Unidecode-*.src.rpm   (* indicates the latest version number) 
    sudo zypper refresh
    rpmbuild --rebuild perl-Text-Unidecode-*.rpm
    cp $HOME/rpmbuild/RPMS/noarch/perl-Text-Unidecode-*.noarch.rpm .
    sudo zypper refresh
    sudo zypper install -y perl-Text-Unidecode-*.noarch.rpm
         
    ```
3. Set environmental variables 
  
   SLES 12     
   ```
   export JAVA_HOME=/usr/lib64/jvm/java-1.7.0-openjdk-1.7.0/bin/      
    ```
   RHEL7/RHEL6     
   ```
   export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.99-2.6.5.0.el7_2.s390x/bin/      
    ```
   Ubuntu 16.04     
   ```
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x     
    ```

4. Download R 3.2.5 source code
    
    ```
    wget https://cran.r-project.org/src/base/R-3/R-3.2.5.tar.gz
    tar zxvf R-3.2.5.tar.gz
    ```
5. Build and Install  R 
   
   ```
   cd R-3.2.5
   ./configure --with-x=no
   make
   sudo make install
   ```
6. Execute test cases 

   RHEL7/ RHEL6/ SLES12     
   ```
    export LANG="en_US.UTF-8"
    make check     
    ```
   Ubuntu 16.04
   ```
   sudo locale-gen "en_US.UTF-8"
   export LANG="en_US.UTF-8"
   make check   
   ```

  
7. Check for version of R-package installed
  
   ```
    echo "sessionInfo()" | R --save
   ```
  
    
 