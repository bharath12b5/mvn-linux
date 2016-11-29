<!---PACKAGE:sysdig--->
<!---DISTRO:SLES 12.x:0.12.1--->
<!---DISTRO:RHEL 7.x:0.12.1--->
<!---DISTRO:Ubuntu 16.x:0.12.1--->


# Building sysdig

The instructions provided below specify the steps to build [sysdig](http://www.sysdig.org) version 0.12.1 on IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10.


_**General Notes:**_ 	 
 * _When following the steps below please use a root user unless otherwise specified._

 * _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Step 1 : Downloading, Building and Installing
1. Install dependencies

    * RHEL 7.1/7.2/7.3
	
			yum install wget tar gcc cmake gcc-c++ make lua-devel.s390x kernel-devel
		
    * SLES 12-SP1/12-SP2
		
			zypper install wget tar gcc cmake make gcc-c++ lua51 lua51-devel kernel-default-devel
		 
	* Ubuntu 16.04/16.10
	
			apt-get install wget tar gcc cmake g++ lua5.1 lua5.1-dev linux-headers-4.4.0-21 linux-headers-4.4.0-21-generic
		
2. Download  Source code
    
        cd  /<source_root>/
        wget https://github.com/draios/sysdig/archive/0.12.1.tar.gz
        tar -xvzf 0.12.1.tar.gz
        cd sysdig-0.12.1
        mkdir build

3. Configure sysdig

        cd  /<source_root>/sysdig-0.12.1/build
        cmake -DUSE_BUNDLED_LUAJIT=OFF ..
		
4.	File changes to be done before build
		
	   Change file `/<source_root>/sysdig-0.12.1/build/driver/Makefile.dkms`
	   at line no `5` to update the Kernel directory.
	
    ```diff
	
-	KERNELDIR  ?= /lib/modules/$(shell uname -r)/build
+	KERNELDIR  ?= /lib/modules/3.10.0-327.18.2.el7.s390x/build
    ```
		
		
	Note : kernel_directory is the one available in `/lib/modules/`
		    In case no kernel_directory is present in `/lib/modules` then create directory in `/lib/modules/` using ' mkdir <dirname> '
			dirname name should match with one which is present in `/usr/src` or `/usr/src/kernels` (RHEL7.1/7.2).
			Once kernel directory is created in `/lib/modules/` perform the below softlink
			
	* RHEL 7.1/7.2/7.3
			
		ln -s /usr/src/kernels/<kernel_directory> /lib/modules/<kernel_directory>/build
				
	* SLES 12-SP2/12-SP2 & Ubuntu 16.04/16.10
			
		ln -s /usr/src/<kernel_directory> /lib/modules/<kernel_directory>/build
			   
		
5. Build and Install sysdig

	    cd  /<source_root>/sysdig-0.12.1/build/	
		make
		make install
		
6. Insert sysdig driver module

         cd  /<source_root>/sysdig-0.12.1/build/driver/
         insmod sysdig-probe.ko

	
## Reference:

http://www.sysdig.org/	
