<!---PACKAGE:Apache ZooKeeper--->
<!---DISTRO:RHEL 6.6:3.4.8--->
<!---DISTRO:RHEL 7.1:3.4.8--->
<!---DISTRO:SLES 11:3.4.8--->
<!---DISTRO:SLES 12:3.4.8--->
<!---DISTRO:Ubuntu 16.x:3.4.8--->
# Building Apache ZooKeeper

Below versions of Apache ZooKeeper are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `3.4.8`

The instructions provided below specify the steps to build Apache ZooKeeper v3.4.8 on Linux on the IBM z Systems for RHEL 6.7/7.1/7.2 and SLES 11-SP3/12/12-SP1.

### Section 1: Install the following dependencies

* For RHEL 7.1/7.2/6.7:

        sudo yum install --nogpgcheck -y git tar wget ant hostname ant-junit cppunit-devel cppunit-doc hamcrest-javadoc hamcrest-demo hamcrest patch xz libtool make
        sudo yum groupinstall -y "Development tools"
   

* For SLES 11-SP3:

        sudo zypper refresh
        sudo zypper install -y git gcc-c++ make tar wget java-1_7_0-ibm java-1_7_0-ibm-devel libcppunit-devel libtool patch 
	                        
* For SLES 12/12-SP1:
   
        sudo zypper install -y git ant ant-junit cppunit-devel cppunit-devel-doc gcc-c++ make autoconf libtool patch wget tar

* Install the following additional dependencies for RHEL6.7 and SLES 11-SP3/12/12-SP1 (Upgrade the version of automake and autoconf)

     * For SLES 11-SP3:
        ```
		   cd /<source_root>/
           wget http://ftp.gnu.org/gnu/m4/m4-1.4.17.tar.xz && tar xf m4-1.4.17.tar.xz
           cd /<source_root>/m4-1.4.17 && ./configure &&  make && sudo make install
		
           cd /<source_root>/
           wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.3-bin.tar.gz
           tar zxf apache-ant-1.9.3-bin.tar.gz    
    
           export ANT_HOME=/<source_root>/apache-ant-1.9.3
           export PATH=$PATH:$ANT_HOME/bin
        ```
		
	 * For SLES 12/12-SP1:
	 
			export ANT_HOME=/usr/share/ant/
	    
    * For SLES 11-SP3 and RHEL6.7:
        ```
			cd /<source_root>/
			wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.xz
			tar xf autoconf-2.69.tar.xz
			cd /<source_root>/autoconf-2.69
			./configure && make && sudo make install
			
			cd /<source_root>/
			wget http://ftp.gnu.org/gnu/automake/automake-1.12.tar.xz
			tar xf automake-1.12.tar.xz
			cd /<source_root>/automake-1.12
			./configure && make && sudo make install && sudo cp automake /usr/bin/
        ```
    
### Section 2: Build and install ZooKeeper
1. Get the source
	```
		cd /<source_root>/
		git clone https://github.com/apache/zookeeper
		cd zookeeper
		git checkout tags/release-3.4.8
	```		

2. Install requirements for Build

			cd /<source_root>/zookeeper/
			ant compile_jute
		
			cd /<source_root>/zookeeper/src/c
			ACLOCAL="aclocal -I /usr/share/aclocal" autoreconf -if

			wget https://issues.apache.org/jira/secure/attachment/12570030/mt_adaptor.c.patch 
			patch src/mt_adaptor.c mt_adaptor.c.patch 
			
			./configure && make && sudo make install
			 sudo make distclean
	
3. Build and test ZooKeeper

			cd /<source_root>/zookeeper/
			ant jar
			ant test    [optional]
* Note: User can ignore intermittent test-case failures as it does not affect the functionality.
        

## References:

        https://zookeeper.apache.org/
