
## Building R 3.2.5

Below versions of R are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `3.2.3`

These instructions describe how to build R(https://www.r-project.org/) for IBM Linux on z system. Version 3.2.5 for RHEL6, RHEL7.1, SLES 12 and Ubuntu 16.04 has been successfully built this way.

**General Notes:**	 
i)  _When following the steps below please use a standard permission user unless otherwise specified._


#### Building R 

1. Installing Build Dependencies

      a) RHEL 7.1

     * With IBM JDK 
 
     ```
     sudo yum install -y wget tar make rpm-build.s390x help2man zlib-devel xz-devel ncurses-devel cairo-devel gcc-c++ libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel xdg-utils pango-devel tcl-devel tk-devel  texinfo gcc-gfortran libXt-devel perl-Text-Unidecode.noarch java-1.7.1-ibm-devel.s390x java-1.7.1-ibm.s390x java-atk-wrapper.s390x javapackages-tools.noarch
     ```
     * With OpenJDK  
     ```
sudo yum install -y wget tar make rpm-build.s390x help2man zlib-devel xz-devel ncurses-devel cairo-devel gcc-c++ libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel xdg-utils pango-devel tcl-devel tk-devel  texinfo gcc-gfortran libXt-devel java-1.7.0-openjdk-devel.s390x perl-Text-Unidecode.noarch

     ```

  b) SLES 12 

     * With IBM JDK 
 
     ```
     sudo zypper install -y wget tar rpm-build help2man zlib-devel xz-devel ncurses-devel make cairo-devel gcc-c++ gcc-fortran libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo java-1_7_1-ibm java-1_7_1-ibm-devel
     ```
     * With OpenJDK  
     ```
     sudo zypper install -y wget tar rpm-build help2man zlib-devel xz-devel ncurses-devel make cairo-devel gcc-c++ gcc-fortran libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo java-1.7.0-openjdk-devel.s390x

     ```
 
  c) RHEL 6
   ```
sudo yum install -y wget tar make rpm-build.s390x help2man zlib-devel xz-devel ncurses-devel cairo-devel gcc-c++ libcurl-devel libjpeg-devel libpng-devel libtiff-devel readline-devel fdupes texlive-helvetic texlive-metafont texlive-psnfss texlive-times xdg-utils pango-devel tcl-devel tk-devel xorg-x11-devel perl-macros texinfo gcc-gfortran libXt-devel java-1_7_0-ibm-devel perl-Text-Unidecode.noarch 
   ```
  b) Ubuntu 16.04

     * With IBM JDK 
     
     Install IBM Java 8

     Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.
       
     ```
     sudo apt-get update
     sudo apt-get install wget tar gcc g++ ratfor  gfortran-4.7  gfortran-4.8 libx11-dev make r-base 
      

     ```
     * With OpenJDK  
     ```
     sudo apt-get update
     sudo apt-get install -y wget tar gcc g++ ratfor  gfortran-4.7  gfortran-4.8 libx11-dev make r-base openjdk-8-jdk

     ```

      
2. Build and install perl-Text-Unidecode (for SLES12) 
   
   Download the latest version of perl-Text-Unidecode 
    ```
    wget http://download.opensuse.org/repositories/devel:/languages:/R:/patched/SLE_12/src/perl-Text-Unidecode-*.src.rpm   (* indicates the latest version number) 
    sudo zypper refresh
    rpmbuild --rebuild perl-Text-Unidecode-*.src.rpm
    cp $HOME/rpmbuild/RPMS/noarch/perl-Text-Unidecode-*.noarch.rpm .
    sudo zypper refresh
    sudo zypper install -y perl-Text-Unidecode-*.noarch.rpm
         
    ```
3. Set environmental variables 
  
   a) SLES 12     
     * With IBM JDK 
 
     ```
export JAVA_HOME=/usr/lib64/jvm/java-1.7.1
     ```
     * With OpenJDK  
     ```
export JAVA_HOME=/etc/alternatives/java_sdk_openjdk


     ```

   b) RHEL 7    
     * With IBM JDK 
 
     ```
export JAVA_HOME=/usr/lib/jvm/java
     ```
     * With OpenJDK  
     ```
export JAVA_HOME=/etc/alternatives/java_sdk_openjdk

     ```

   c) RHEL 6    
 
     ```
export JAVA_HOME=/usr/lib/jvm/java
     ```
   d) Ubuntu 16.04
    
     * With IBM JDK 
 
     ```
export JAVA_HOME=/opt/ibm/java-s390x-80
     ```
     * With OpenJDK  
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
  
    
 