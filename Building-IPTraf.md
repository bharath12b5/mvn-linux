#Building IPTraf

Below version of IPTraf is available in respective distributions at the time of this recipe creation:

*    RHEL 6.8 has `3.0.1`
*    RHEL 7.1 has `1.1.4`
*    RHEL 7.2 has `1.1.4`
*    SLES 11 has `3.0.0`
*    Ubuntu 16.04 has `3.0.0`

The instructions provided below specify the steps to build IPTraf version 3.0.0 on Linux on the IBM z Systems for RHEL 7.1/7.2, SLES 12/12-SP1.

_**General Notes:**_

_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._


###1. Install build dependencies

For RHEL 7.1/7.2

	sudo yum install gcc tar git wget ncurses-devel ncurses make

For SLES 12-SP1

	sudo zypper install gcc tar git wget ncurses-devel ncurses make

For SLES 12

	sudo zypper install  gcc tar git wget ncurses-devel libncurses5 make

###2. Download and extract IPTraf source tarball

	cd /<source_root>/
	wget ftp://iptraf.seul.org/pub/iptraf/iptraf-3.0.0.tar.gz
	tar zxvf iptraf-3.0.0.tar.gz
				
###3. Build and install IPTraf  

	cd /<source_root>/iptraf-3.0.0
	sudo cp /usr/include/netinet/if_tr.h /usr/include/linux/
	sudo ./Setup

###4. Set PATH variable

	export PATH=$PATH:/usr/local/bin
		
###5. Run IPTraf using the command below

	sudo /usr/local/bin/iptraf
		
##References:  
http://iptraf.seul.org/