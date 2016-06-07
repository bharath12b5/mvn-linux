<!---PACKAGE:Ruby--->
<!---DISTRO:SLES 12:Distro,2.3--->
<!---DISTRO:SLES 11:Distro,2.3--->
<!---DISTRO:RHEL 7.1:Distro,2.3--->
<!---DISTRO:RHEL 6.6:Distro,2.3--->
<!---DISTRO:Ubuntu 16.x:Distro,2.3--->

# Building Ruby

Below versions of Ruby are available in respective distributions at the time of this recipe creation:

*    RHEL 7.1 has `2.0.0p598`
*    RHEL 6.6 has `1.8.7p374`
*    SLES 12 has `2.1.2p95`
*    SLES 11.3 has `1.8.7p357`
*	 UBUNTU 16.04 has `2.3.0p0`

The instructions provided below specify the steps to build Ruby version 2.3.1 on Linux on the IBM z Systems for RHEL 6.6/7.1, SLES 11/12, UBUNTU 16.04.

Packages that use ruby may require a specific version, in particular, having [RubyGems](http://en.wikipedia.org/wikiRubyGems) is essential for large packages using Ruby (Ruby version >= 1.9) The authoritative source for most things "ruby" is [here](http://www.ruby-lang.org) for the latest "stable" Ruby version see [download link](http://www.ruby-lang.org/en/downloads)

_**General Notes:**_

_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### 1.   Install the dependencies for your specific platform

#### RHEL 6.6
    
```sh
    sudo yum install bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel gcc make wget tar bzip2 svn
```


#### RHEL 7.1
    
```sh
    sudo yum install bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel gcc make wget tar 
```

#### SLES 12
    
```
    sudo zypper install bison flex libopenssl-devel libyaml-devel libffi48-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel gcc make wget tar
```
#### SLES 11
```
    sudo zypper install bison flex libopenssl-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel gcc make wget tar bzip2 subversion
```
#### UBUNTU 16.04

```
	sudo apt-get install -y gcc make wget tar bzip2 subversion bison flex openssl
```
	
### 2.   Download and Install the GCC 6.0.0 (Only for RHEL6.6 and SLES11)
   To build the GCC 6.0.0, refer the "Building and Installing GCC" part of the [GCCGO recipe](https://github.com/linux-on-ibm-z/docs/wiki/Building-gccgo?cm_mc_uid=60971096199214062909410&cm_mc_sid_50200000=1438603503)



### 3.   Download and unpack the Ruby 2.3.1 source code
```
    cd /<source_root>/
    wget http://cache.ruby-lang.org/pub/ruby/ruby-2.3.1.tar.gz
    tar zxf ruby-2.3.1.tar.gz
    cd ruby-2.3.1
```
### 4.   Configure and build Ruby 2.3.1
```
    ./configure
    make
```
### 5.    Test the results (optional) and install ruby
```
    make test
```
   _**Note:** The tests should not report any failures_

  
  
```
    sudo -E make install
```


  _The default location for installation when building from source is in /usr/local -- if ruby was 
    installed not by building from source, it would be in /usr -- there are other ways to have 
    multiple versions of ruby (or other packages installed) but taking the default is the simplest -- 
    when installing things that require the from source version of ruby, make sure the appropriate 
    path (/usr/local/bin) is first in the PATH environment variable._

### 6.   Verify that ruby is available
```
    ruby -v
    gem env
```
   _**Note:** Rubygems support is contained as part of this build, so `gem env` should work_
