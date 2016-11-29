# Building PostgreSQL

Below versions of PostgreSQL are available in respective distributions at the time of this recipe creation:  

* RHEL 7.1 & 7.2 have 9.2.15  
* RHEL 7.3 has 9.2.18
* RHEL 6.8 has  8.4.20    
* SLES 11-SP4 has 8.3.23-0.4.1  
* SLES 12 & SLES 12-SP1 have 9.4.6-7.2	 
* Ubuntu 16.04 has  9.5+173  
* Ubuntu 16.10 has  9.5.5 

[PostgreSQL version 9.6.0](http://www.postgresql.org/) has been successfully built and tested for Linux on z Systems. The following instructions can be used for SLES 11-SP4/12/12-SP1 and RHEL 6.8/7.1/7.2/7.3 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	 
- _When following the steps below please use a standard permission user unless otherwise specified._

- _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._ 

1. Install the build dependencies:

      (SLES 11-SP4/12/12-SP1)
     ```
    sudo zypper in -y git gcc  gcc-c++ make readline-devel  zlib-devel bison flex
    ```

       (RHEL 6.8/7.1/7.2/7.3)
      ```
    sudo yum install -y git wget build-essential gcc gcc-c++ make readline-devel zlib-devel bison flex
      ```
	  
	  (Ubuntu 16.04/16.10)
      ```
    sudo apt-get update
    sudo apt-get install -y  bison flex wget build-essential git gcc make zlib1g-dev libreadline6 libreadline6-dev
      ```
	  
	  
2. Create postgres user, group and home directory:
 	
     ```
    sudo groupadd -r postgres 
	sudo useradd -r -m -g postgres postgres
	sudo passwd postgres
     ```
	 
    **Note:** *Password for SLES11-SP4 should be up to 7 characters. Please note that `/usr/sbin` is available in `PATH` environment variable.*
	
3. Build and install PostgreSQL:
 	  * Download PostgreSQL source code from github:
  
         ```
         su postgres -s /bin/bash
         cd /home/postgres
         git clone https://github.com/postgres/postgres.git 
         cd postgres/
         git checkout REL9_6_0
         ```   
     
   		 **Note:** *Login as postgres user and download the source in postgres home directory.* 

	  * Configure, build and test the build:
   
       ```
       ./configure
       make
       make check
       ``` 

		 **Note:** *`make check` won't work as root; do it as an unprivileged user.*
    
   * Install PostgreSQL as a standard permission user:

       ```
       su <standard_permission_user>
       sudo make install
	   export PATH=$PATH:/usr/local/pgsql/bin
       ``` 
	 **Note:** *Please make sure the postgres user directory has sufficient read and execute permissions when executing.* 
	   
4. Set up PostgreSQL server: 
    * Create PostgreSQL data directory to store data and make postgres user as the owner:
	
        ```
        sudo mkdir /usr/local/pgsql/data 
        sudo chown postgres:postgres /usr/local/pgsql/data
        ```
    * Initialize PostgreSQL data directory as postgres user:
	
        ```
        su postgres -s /bin/bash
        /usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data/
        ```	
     **Note:** *Please make sure the directory `/usr/local/` has sufficient read and execute permissions when initializing.*   
       
    * Start the PostgreSQL server:
        ```
        /usr/local/pgsql/bin/postmaster -D /usr/local/pgsql/data
		```       


### References:
https://github.com/postgres/postgres.git