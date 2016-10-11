<!---PACKAGE:Xerces-C--->
<!---DISTRO:SLES 12:3.1.4--->
<!---DISTRO:SLES 11:3.1.4--->
<!---DISTRO:RHEL 7.1:3.1.4--->
<!---DISTRO:RHEL 6.6:3.1.4--->
<!---DISTRO:Ubuntu 16.x:3.1.4--->

#Building Xerces-C 3.1.4

Below version of Xerces is available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `3.1.3`

The instructions provided below specify the steps to build Xerces-C 3.1.4 on Linux on the IBM z Systems for RHEL 6.6/7.1, SLES 11/12 and Ubuntu 16.04. More information on Xerces-C is available at https://xerces.apache.org/xerces-c/ and the source code can be downloaded from https://github.com/apache/xerces-c.

_**General Notes:**_ 	

_i) When following the steps below please use a standard permission user unless otherwise specified._
	 
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Building Xerces-C

### Obtain pre-built dependencies

1. Use the following commands to obtain dependencies

    For RHEL 6.6

    ```shell
    sudo yum install git automake libtool make gcc-c++ tar
    ```
	
	For RHEL 7.1

    ```shell
    sudo yum install git automake libtool make gcc-c++
    ```
	
    For SLES 11
	
    ```shell
    sudo zypper install gcc-c++ make git-core tar wget m4
    ```
	
	For SLES 12
	
    ```shell
    sudo zypper install libtool automake gcc-c++ make git-core
    ```
	
	For Ubuntu 16.04
	
    ```shell
	sudo apt-get update
    sudo apt-get install libtool automake g++ make git-core
    ```
	
    
2. Create the `/<source_root>/` as mentioned above

   ```shell
      mkdir /<source_root>/
   ```
   
### Dependency Builds: libtool, autoconf, and automake - For SLES11 only
 
_**Note:** These Dependency Builds for later/latest versions must be carried out in the order given in this recipe._

#### Install libtool 2.4.6

  1. Download libtool source code
	 
     ```shell
	 cd /<source_root>/
	 mkdir libtool
     cd libtool
     wget http://ftpmirror.gnu.org/libtool/libtool-2.4.6.tar.gz
     tar -xvf libtool-2.4.6.tar.gz
     cd libtool-2.4.6
     ```

  2. Configure and make
	
     ```shell
     ./configure
     make
     ```
  3. Install libtool

     ```shell
     sudo make install
	 ```
	
#### Install autoconf 2.69 

  1. Download autoconf source code

     ```shell
	 cd /<source_root>/
	 mkdir autoconf
     cd autoconf
     wget http://ftpmirror.gnu.org/autoconf/autoconf-2.69.tar.gz  
     tar -xvf autoconf-2.69.tar.gz  
     cd autoconf-2.69
     ```

  2. Configure and make

     ```shell
     ./configure
     make
     ```
  3. Install autoconf
 
     ```shell
     sudo make install
	 ```

#### Install automake 1.13.4 

  1. Download automake source code

     ```shell
	 cd /<source_root>/
	 mkdir automake
     cd automake
     wget http://ftpmirror.gnu.org/automake/automake-1.13.4.tar.gz   
     tar -xvf automake-1.13.4.tar.gz   
     cd automake-1.13.4
     ```

  2. Configure and make

    ```shell
     ./configure
     make
     ```
  3. Install automake

     ```shell
     sudo make install
	 ```

### Product Build - Xerces-C

1. Download Xerces-C 3.1.4 source code

	 ```shell
	 cd /<source_root>/
     git clone https://github.com/apache/xerces-c.git 
     cd xerces-c
	 git checkout Xerces-C_3_1_4 
     ```
1. Set Environment variables

	For RHEL 6.6/7.1
	
	_**Note:** On RHEL7.1, if the command `locale -a` does not return a list including 'en_US.UTF-8' then try try to reload  glibc-common using the command `sudo yum reinstall glibc-common`._ 

    ```shell
    export LANG='en_US.UTF-8'
    export LC_ALL='en_US.UTF-8'
    ```
	
	For SLES 11/12

    ```shell
    export LANG='en_US.UTF-8'
    export LANGUAGE='en_US.UTF-8'
    ```
	
	For Ubuntu 16.04

    ```shell
    export LANG='en_US.UTF-8'
	export LANGUAGE='en_US.UTF-8'
    ```

1. Configure locale 'en_US.UTF-8' on Ubuntu 16.04
    ```shell
    sudo locale-gen en_US en_US.UTF-8
	sudo dpkg-reconfigure locales 
	```
	
1. Configure and make

    ```shell
	./reconf
    ./configure
     make
    ```
	
	Generate the configuration script using reconf. Configure the environment using configure and compile the files using make.
	
1. Check the make

     ```shell
     make check
     ```

	 Run the test suite using make check . It generates test-results.log,the output of smoketest which is compared with an expected results log under ./scripts/sanityTest_ExpectedResult.log. If both match, it indicates that the testing passed.
	 
1. Install Xerces-C

    ```shell
    sudo make install
    ```
	
	Install Xerces-C using make install. As Xerces-C is an XML parser it has no specific command line. It has libraries which you can find in /usr/local/lib and /usr/local/include of the machine.
	
1. Verification

    Under /usr/local/lib path you can find libs like libxerces-c-3.1.so, libxerces-c.a, libxerces-c.la, libxerces-c.so and directory pkgconfig.
	Under /usr/local/include/xercesc the following folders would be present.
	dom  framework  internal  parsers  sax  sax2  util  validators  xinclude
	
	
###References:

https://xerces.apache.org/xerces-c/ - Details of Xerces-C can be found here.
