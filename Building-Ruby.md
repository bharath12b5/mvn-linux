<!---PACKAGE:Ruby--->
<!---DISTRO:SLES 12:Distro,2.3--->
<!---DISTRO:SLES 11:Distro,2.3--->
<!---DISTRO:RHEL 7.1:Distro,2.3--->
<!---DISTRO:RHEL 6.6:Distro,2.3--->
<!---DISTRO:Ubuntu 16.x:Distro,2.3--->

# Building Ruby

Below versions of Ruby are available in respective distributions at the time of this recipe creation:

*    RHEL 6.8 has `1.8.7p374`
*    RHEL 7.1 has `2.0.0p598`
*    RHEL 7.2/7.3 has `2.0.0.648`
*    SLES 12 has `2.1.2p95`
*    SLES 12-SP1/12-SP2 has `2.1-1.6`
*    SLES 11.4 has `1.8.7p357`
*    Ubuntu 16.04/16.10 has `2.3.1p112`

The instructions provided below specify the steps to build Ruby version 2.3.3 on IBM z Systems for RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2, Ubuntu 16.04/16.10.

_**General Notes:**_

_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Prerequisites 

* RHEL 6.8/SLES 11-SP4

    * GCC 6.0.0
      
	  Instructions for building GCC can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-gccgo?cm_mc_uid=60971096199214062909410&cm_mc_sid_50200000=1438603503)

### 1.   Install dependencies

* RHEL 6.8
    
```sh
sudo yum install bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel gcc make wget tar bzip2 svn
```


* RHEL 7.1/7.2/7.3
    
```sh
sudo yum install bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel gcc make wget tar
```

* SLES 12/12-SP1/12-SP2
    
```
sudo zypper install bison flex libopenssl-devel libyaml-devel libffi48-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel gcc make wget tar
```
* SLES 11-SP4 

```
sudo zypper install bison flex libopenssl-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel gcc make wget tar bzip2 subversion
```
* Ubuntu 16.04/16.10

```
sudo apt-get install -y gcc make wget tar bzip2 subversion bison flex openssl
``` 
	
### 2. Download Ruby 2.3.3 source code 
```sh
cd /<source_root>/
wget http://cache.ruby-lang.org/pub/ruby/ruby-2.3.3.tar.gz
tar zxf ruby-2.3.3.tar.gz
cd ruby-2.3.3
```
### 3. Configure and build Ruby 2.3.3 
```sh
./configure
make
```
### 4. Install Ruby 2.3.3 
```sh
sudo -E make install
```
   _**Note:** Set PATH variable to `/usr/local/bin` while installing Ruby from source_

### 5. Test the results (optional) 
```sh
make test
```
   _**Note:** The tests should not report any failures_


### 6. Verify Ruby 2.3.3 
```
ruby -v
gem env
```
   _**Note:** Rubygems support is contained as part of this build, so `gem env` should work_

### References

https://github.com/ruby/ruby  
http://www.ruby-lang.org  
http://www.ruby-lang.org/en/downloads  
http://en.wikipedia.org/wikiRubyGems  
