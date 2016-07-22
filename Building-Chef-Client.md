<!---PACKAGE:Chef Client--->
<!---DISTRO:SLES 12:12.12--->
<!---DISTRO:SLES 11:12.12--->
<!---DISTRO:RHEL 7.1:12.12--->
<!---DISTRO:RHEL 6.6:12.12--->
<!---DISTRO:Ubuntu 16.x:12.12--->

## Building Chef Client 12.12.13

The instructions provided below specify the steps to build Chef Client 12.12.13 on Linux on the IBM z Systems for RHEL 6.6/7.1, SLES 11/12 and Ubuntu 16.04.

_**General Notes:**_   
_i) When following the steps below please use a standard permission user unless otherwise specified._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._


## Building Chef Client on RHEL 7.1/6.6, SLES 12/11, UBUNTU 16.04

1. Install the build dependencies

    For RHEL 7.1 
    ```
    sudo yum install -y git gcc make wget tar bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel which
    ```
	
    For RHEL 6.6 
    ```
    sudo yum -y install git gcc make wget tar bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel which	  
    ```
    
    For SLES 11
    ```
    sudo zypper install -y git gcc make wget tar bison flex libopenssl-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel	      
    ```

    For SLES 12
    ```
    sudo zypper install -y git gcc make wget tar bison flex libopenssl-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel
    ```
	
	For Ubuntu 16.04
	```
	sudo apt-get install -y git make gcc make wget build-essential zlib1g zlib1g-dev libssl-dev libreadline-dev libgdbm-dev
	```

2. For SLES 11 you will need to build Openssl  

    ```
    cd /<source_root>/
    wget ftp://openssl.org/source/openssl-1.0.2h.tar.gz
    tar zxf openssl-1.0.2h.tar.gz
    cd openssl-1.0.2h
    ./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib shared zlib-dynamic
    make
    sudo make install
    ```
    
3. Build Ruby 2.2.4
   
   3.1. Download the source code
   ```
   cd /<source_root>/
   wget http://cache.ruby-lang.org/pub/ruby/ruby-2.2.4.tar.gz
   tar zxf ruby-2.2.4.tar.gz
   cd ruby-2.2.4
   ```
	
  For SLES 11
  ```
  ./configure LDFLAGS='-L/<source_root>/openssl-1.0.2h' --with-openssl-include=/<source_root>/openssl-1.0.2h/include --with-openssl-dir=/usr/
  ```
	  
  For RHEL 6.6, RHEL 7.1, SLES12 and Ubuntu 16.04
  ```
  ./configure
  ```

   3.2. Build the source code
   ```
   make
   make test	  
   sudo make install
   ```
	
4. Move to the location you wish to store the Chef source in

    ```
    cd /<source_root>/
    ```

5. Clone the github Chef client repository checkout the correct version

    ```
    git clone https://github.com/chef/chef.git
    cd chef
    git checkout v12.12.13
    ```

6. Correct the gem environment for a standard user

    ```
    export GEM_HOME=/home/<USER>/.gem/ruby
    export PATH=/home/<USER>/.gem/ruby/bin:$PATH
    ``` 

    _where `<USER>` is the standard user you are installing under._
       
   _**Note**: Run ```gem env``` to verify the state of the environment, if later on you have issues installing / running ruby gems please ensure the environment is set correctly._
	
7. Install the required version of the bundler ruby gem

   ```
   gem install bundler -v '1.11.2'
   ```
	
8. Use bundler to install Chef Client's ruby gem dependencies

   ```
   bundle install
   ```

9. Build the Chef Client ruby gem packages

   ```
   bundle exec rake gem
   ```

   _**NOTE:** For RHEL7, if ```bundler: command not found: rake``` error occurs. Set ```rake``` binary path to PATH environment variable and rerun above command._

    ```
      export PATH=$PATH:$HOME/bin
    ```

10. Install the gem you just built

   ```
   ls pkg/*.gem | grep -v mingw32 | xargs gem install
   ```    
   
11. Chef client is now built and installed 

   ```
   chef-client -version
   ```    
   
## Testing Chef Client

If you'd like to test the Chef client you've just built and installed, just follow the steps below:

1. Set CHEF_FIPS variable
	
	```
	export CHEF_FIPS=""
	```

2. Update Fileutils method for Ubuntu 16.04

	```
	cd /<source_root>/chef/
	sed -i "s/remove_entry_secure(@repository_dir, force = Chef::Platform.windows?)/rmdir(@repository_dir)/g" ./spec/support/shared/integration/integration_helper.rb
	sed -i "s/rm_r/rmdir/g" ./spec/functional/run_lock_spec.rb
	```

3. Run the test suite
   	
   ```
   bundle exec rake spec
   ```
   
   To run a single test file
   ```  
   bundle exec rspec spec/PATH/TO/FILE_spec.rb
   ```  
   To run a subset of tests
   ```
   bundle exec rspec spec/PATH/TO/DIR
   ```
   
   _**NOTE:**_
   There could be a test case failure "the cheffish DSL tries to load but fails (because chef-provisioning is not there)" which can be ignored as this is not specific to s390x.


4. Visit https://github.com/chef/chef#testing for more   

## References:

https://github.com/chef/chef.git