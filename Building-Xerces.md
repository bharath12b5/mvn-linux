<!---PACKAGE:Xerces-C--->
<!---DISTRO:SLES 12:3.1--->
<!---DISTRO:SLES 11:3.1--->
<!---DISTRO:RHEL 7.1:3.1--->
<!---DISTRO:RHEL 6.6:3.1--->
<!---DISTRO:Ubuntu 16.x:3.1--->

#Building Xerces-C 3.1

Below version of Xerces is available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `3.1.1`

The instructions provided below specify the steps to build Xerces-C 3.1 on Linux on the IBM z Systems for RHEL 6/7 and SLES 11/12. More information on Xerces-C is available at https://xerces.apache.org/xerces-c/ and the source code can be downloaded from https://github.com/apache/xerces-c.

_**General Notes:**_ 	

i) When following the steps below please use a standard permission user unless otherwise specified._
	 
ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Building Xerces-C

### Obtain pre-built dependencies

1. Use the following commands to obtain dependencies

    For RHEL 6.6

    ```shell
    yum install git automake libtool make gcc-c++ tar
    ```
	
	For RHEL 7.1

    ```shell
    yum install git automake libtool make gcc-c++
    ```
	
    For SLES 11
	
    ```shell
    zypper install gcc-c++ make git-core tar wget m4
    ```
	
	For SLES 12
	
    ```shell
    zypper install libtool automake gcc-c++ make git-core
    ```
    
2. Create the `/<source_root>/` as mentioned above.

   ```shell
      mkdir /<source_root>/
   ```
   
### For SLES11 only
### Dependency Build - libtool 2.4.6 - Only for SLES 11

_**Note:** These Dependency Builds to libtool, autoconf, and automake for later/latest versions must be carried out in the order given in this recipe._


  1. Download libtool source code
	 
	 For SLES 11
	 
     ```shell
	 cd /<source_root>/
	 mkdir libtool
     cd libtool
     wget http://ftpmirror.gnu.org/libtool/libtool-2.4.6.tar.gz
     tar -xvf libtool-2.4.6.tar.gz
     cd libtool-2.4.6
     ```

  2. Configure and make

	For SLES 11
	
     ```shell
     ./configure
     make
     ```
  3. Install libtool
  
    For SLES 11
     ```shell
     sudo make install
	 ```
	
### Dependency Build - autoconf 2.69 - Only for SLES 11

  1. Download autoconf source code
	 
	 For SLES 11
	 
     ```shell
	 cd /<source_root>/
	 mkdir autoconf
     cd autoconf
     wget http://ftpmirror.gnu.org/autoconf/autoconf-2.69.tar.gz  
     tar -xvf autoconf-2.69.tar.gz  
     cd autoconf-2.69
     ```

  2. Configure and make

	For SLES 11
	
     ```shell
     ./configure
     make
     ```
  3. Install autoconf
  
    For SLES 11
     ```shell
     sudo make install
	 ```

### Dependency Build - automake 1.13.4 - Only for SLES 11

  1. Download automake source code
	 
	 For SLES 11
	 
     ```shell
	 cd /<source_root>/
	 mkdir automake
     cd automake
     wget http://ftpmirror.gnu.org/automake/automake-1.13.4.tar.gz   
     tar -xvf automake-1.13.4.tar.gz   
     cd automake-1.13.4
     ```

  2. Configure and make

	For SLES 11
	
     ```shell
     ./configure
     make
     ```
  3. Install automake
  
    For SLES 11
     ```shell
     sudo make install
	 ```

### Product Build - Xerces-C

1. Download Xerces-C 3.1 source code

	 ```shell
	 cd /<source_root>/
     git clone https://github.com/apache/xerces-c.git --branch xerces-3.1
     cd xerces-c
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
