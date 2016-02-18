# Building PostgreSQL

[PostgreSQL version 9.5](http://www.postgresql.org/) has been successfully built and tested for Linux on z Systems. The following instructions can be used for SLES 11/12 and RHEL 6.6/7.1.

_**General Notes:**_ 	 
_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._ 

1. Install the build dependencies:

      (SLES 11/12)
     ```
    sudo zypper in -y git gcc gcc-c++ make readline-devel zlib-devel bison flex
    ```

       (RHEL 6.6/7.1)
      ```
    sudo yum install -y git wget build-essential gcc gcc-c++ make readline-devel zlib-devel bison flex
      ```
2. Create postgres user, group and home directory:
 	
    RHEL 6.6/7.1 ,SLES 11/12 
     ```
    sudo groupadd -r postgres 
	sudo useradd -r -m -g postgres postgres
	sudo passwd postgres
     ```
    
	 **Note:** *Password for SLES11 should be up to 7 characters.Please note that `/usr/sbin` is available in `PATH` environment variable.*
	 
3. Build and install PostgreSQL:
 	  * Download PostgreSQL source code from github:
  
         ```
         su postgres -s /bin/bash
         cd /home/postgres
         git clone https://github.com/postgres/postgres.git 
         cd postgres/
         git checkout REL9_5_0
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
       cd /home/postgres/postgres
       sudo make install
	   export PATH=$PATH:/usr/local/pgsql/bin
       ``` 
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
     **Note:** *Please make sure the directory `/usr/local/` has sufficent read and execute permissions when initializing.*   
       
    * Start the PostgreSQL server:
        ```
        /usr/local/pgsql/bin/postmaster -D /usr/local/pgsql/data
		```       


### References:
https://github.com/postgres/postgres.git