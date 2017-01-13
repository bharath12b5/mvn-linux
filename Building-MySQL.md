<!---PACKAGE:MySQL--->
<!---DISTRO:SLES 12:5.7--->
<!---DISTRO:SLES 11:5.7--->
<!---DISTRO:RHEL 7.1:5.7--->
<!---DISTRO:RHEL 6.6:5.7--->
<!---DISTRO:Ubuntu 16.x:Distro, 5.7--->

## Building MySQL

Below versions of MySQL are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `5.7.12-0ubuntu1`
*    SLES 11-SP3 has `5.5.45-0.11.1`
*    RHEL 6.6 has `5.1.73`

The instructions provided below specify the steps to build MySQL version 5.7.17 on Linux on the IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

More information on MySQL is available at https://www.mysql.com and the source code can be downloaded from https://github.com/mysql/mysql-server.git.

_**General Notes:**_

i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Building MySQL

###Obtain pre-built dependencies

*	For RHEL 6.8

	```shell
	sudo yum install git gcc gcc-c++ make cmake bison ncurses-devel util-linux tar zip wget zlib-devel bzip2.s390x libtool.s390x
	```
*	For RHEL 7.1/7.2/7.3

	```shell
	sudo yum install git gcc gcc-c++ make cmake bison ncurses-devel perl-Data-Dumper
	```
*	For SLES 11-SP4 - _(Additional support packages are needed to update cmake)_

    ```shell
    sudo zypper install git gcc gcc-c++ make cmake bison ncurses-devel util-linux tar zip wget glibc-devel-32bit zlib-devel
    ```
*	For SLES 12/12-SP1/12-SP2

	```shell
	sudo zypper install git gcc gcc-c++ make cmake bison ncurses-devel wget tar
	```
*	For Ubuntu 16.04/16.10

	```shell
	sudo apt-get update
	sudo apt-get install git make cmake gcc g++ libncurses5-dev bison
	```

###Dependency Build - GCC 4.8.2 and cmake 3.3.0  

* Install GCC 4.8.2 by building from source. (Required on RHEL 6.8 and SLES 11-SP4)
      
  * Download the GCC source code, then extract it.

    ```shell
    cd /<source_root>/
    wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.bz2
    bunzip2  gcc-4.8.2.tar.bz2
    tar xvf gcc-4.8.2.tar
  ```

  * Download the prerequisites needed for GCC.

  ```shell
  cd gcc-4.8.2/
  ./contrib/download_prerequisites
  mkdir build
  cd build
  ```	
  
  * Configure the Makefile. Then Make and Install the utility.

  ```shell
  ../configure --disable-multilib --disable-checking --enable-languages=c,c++ --enable-multiarch --enable-shared --enable-threads=posix --without-included-gettext --with-system-zlib --prefix=/opt/gcc4.8
  make 
  sudo make install 
  ```
  * Set the environment variable.

  ```shell
  export PATH=/opt/gcc4.8/bin:$PATH
  export LD_LIBRARY_PATH=/opt/gcc4.8/lib64/
  ```

  * Confirm the version of `GCC`

 ```shell
 gcc --version
 ```
      
* Update cmake to version 3.3.0 by building from source. (Required on SLES 11-SP4 only)

  * _[Optional]_ Check the version of any existing `cmake` executable.

 ```shell
 which cmake
 $(which cmake) --version
 ```
   _**Note:** A `cmake` at version 2.6.3 or later should be usable without upgrade._
   
  * Download the cmake source code, then extract it

 ```shell
 cd /<source_root>/
 wget http://www.cmake.org/files/v3.3/cmake-3.3.0.tar.gz
 tar xzf cmake-3.3.0.tar.gz
 ```

  * Bootstrap to configure the Makefile. Then Make and Install the utility

 ```shell
 cd cmake-3.3.0
 ./bootstrap --prefix=/usr
 gmake
 sudo gmake install -e LD_LIBRARY_PATH=/opt/gcc4.8/lib64/
 ```
   _**Note:** To place `cmake` in the standard SLES location use `./bootstrap --prefix=/usr`._

  * Confirm the location and version of the upgraded `cmake`

 ```shell
 which cmake
 $(which cmake) --version
 ```
	  
###Product Build - MySQL

   1. Download the MySQL source code from Github
    ```shell
    cd /<source_root>/
    git clone https://github.com/mysql/mysql-server.git
    ```

   2. Move into the ` mysql-server` sub-directory, and checkout 5.7.17
    ```shell
    cd mysql-server
    git branch
    git checkout tags/mysql-5.7.17
    ```

   3. Configure and Build the MySQL
    
	For RHEL 6.8
	```shell
    cmake -DCMAKE_C_COMPILER=/opt/gcc4.8/bin/gcc -DCMAKE_CXX_COMPILER=/opt/gcc4.8/bin/g++ -DDOWNLOAD_BOOST=1 -DWITH_BOOST=.
    gmake
    ```
	
	For RHEL 7.1/7.2/7.3
	```shell
    cmake . -DDOWNLOAD_BOOST=1 -DWITH_BOOST=.
    gmake
    ```
	
	For Ubuntu 16.04/16.10
	```shell
    cmake . -DDOWNLOAD_BOOST=1 -DWITH_BOOST=.
    make
    ```
	
	For SLES 11-SP4
	```shell
	wget http://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
	tar xzf boost_1_59_0.tar.gz
    cmake -DCMAKE_C_COMPILER=/opt/gcc4.8/bin/gcc -DCMAKE_CXX_COMPILER=/opt/gcc4.8/bin/g++ -DDOWNLOAD_BOOST=1 -DWITH_BOOST=.
    gmake
    ```
	
	For SLES 12/12-SP1/12-SP2
	```shell
	wget http://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz 
	tar xzf boost_1_59_0.tar.gz 
	cmake . -DDOWNLOAD_BOOST=1 -DWITH_BOOST=.
	gmake
    ```

   4. _[Optional]_ Run unit tests

    The testing should take only a few seconds.

    For RHEL6.8, RHEL 7.1/7.2/7.3 , SLES 11-SP4 & SLES 12/12-SP1/12-SP2
	```shell
    gmake test
    ```
	
	For Ubuntu 16.04/16.10
	```shell
    make test
    ```
	
   5. Install MySQL into the standard location

	For RHEL 7.1/7.2/7.3 & SLES 12/12-SP1/12-SP2
	```shell
    sudo gmake install
    ```
	
	For RHEL 6.8 & SLES 11-SP4
	```shell
    sudo gmake install -e LD_LIBRARY_PATH=/opt/gcc4.8/lib64/
    ```
	
	For Ubuntu 16.04/16.10
	```shell
    sudo make install
    ```

###_[Optional]_ Post installation Setup and Testing

   Refer http://dev.mysql.com/doc/refman/5.7/en/postinstallation.html for the post installation Setup and Testing steps.

###_[Optional]_ Clean up

   1. Remove the ` /<source_root>/` directory to tidy up.

     ```shell
     cd /<source_root>/
     cd ..
     sudo rm -rf /<source_root>/
     ```

###References:

https://bugs.mysql.com/bug.php?id=72752 - Explanation of the cmake upgrade for SLES 11.

http://www.mysql.com - MySQL Homepage with definitive Information and Documentation and configuration for non-test environments.



