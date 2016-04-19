<!---PACKAGE:Fluentd--->
<!---DISTRO:SLES 12:0.12.20--->
<!---DISTRO:SLES 11:0.12.20--->
<!---DISTRO:RHEL 7.1:0.12.20--->
<!---DISTRO:RHEL 6.6:0.12.20--->
<!---DISTRO:Ubuntu 16.x:0.12.20--->

# Building Fluentd

Fluentd version 0.12.20 has been successfully built and tested for Linux on z Systems.  The following instructions can be used for RHEL 7.1/6.6, SLES 12/11 and Ubuntu 16.04.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._  

iii) _At the time of writing recipe, the version of Fluentd was 0.12.20._   

1. Install the dependencies for your specific platform

    RHEL 7.1:
    ```shell
    sudo yum install git ruby gcc gcc-c++ kernel-devel ruby-devel openssl make tzdata
    ```
    RHEL 6.6:
    ```shell
    sudo yum install bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel gcc make wget tar tzdata
    ```
    SLES 12:
    ```shell
    sudo zypper install bison flex libopenssl-devel libyaml-devel libffi48-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel gcc make wget tar timezone
    ```
    SLES 11:
    ```shell
    sudo zypper install bison flex libopenssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel gcc make wget tar timezone
    ```
	Ubuntu 16.04:
	```
	sudo apt-get install -y ruby ruby-dev gcc make
	```
2. For platforms other than RHEL 7.1 and Ubuntu 16.04, build Ruby 2.3.0 from the instructions [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby)

    Once the Ruby build is completed and installed continue with the instructions below

3. For platforms other than RHEL 7.1 and Ubuntu 16.04, setup the gem HOME and PATH environment for a standard user

    ```shell
    export GEM_HOME=/home/<USER>/.gem/ruby
    export PATH=/home/<USER>/.gem/ruby/bin:$PATH
    ```  
    Where `<USER>` is the standard user you are logged in as.
	
4. Build & install the latest stable fluentd gem and dependencies

    Fluentd is a gem based product, so the rubygem install process will automatically build the necessary native parts.
	
	For RHEL 6.6 and SLES:  
    ```shell
    gem install fluentd
    ```
	
	For RHEL 7.1 and Ubuntu 16.04:
	```shell
    sudo gem install fluentd
    ```
	
    This will lead to output such as
    ```shell
    Fetching: string-scrub-0.0.5.gem (100%)
    Building native extensions.  This could take a while...
    Successfully installed string-scrub-0.0.5
    Fetching: thread_safe-0.3.5.gem (100%)
    Successfully installed thread_safe-0.3.5
    ....................
    Parsing documentation for fluentd-0.12.20
    Installing ri documentation for fluentd-0.12.20
    Done installing documentation for string-scrub, thread_safe, tzinfo, tzinfo-data, sigdump, http_parser.rb, cool.io, yajl-ruby, msgpack, fluentd after 13 seconds
    10 gems installed
    ```
    Once complete it should report that a number of gems including fluentd are installed. Verify the installed version with `gem list fluentd`.  Note that installation of different versions of fluentd can be done with `gem install fluentd -v '0.12.8'` for example.

5. _**Optional:**_ Quick test of the fluentd installation.

	This test is to setup/install a config directory, then start a fluent process which is put into the background.  Finally a message is piped to `fluent-cat`.
  
  ```shell
  fluentd -s conf
  fluentd -c conf/fluent.conf &
  echo '{"json":"message"}' | fluent-cat debug.test
  ```
  This should display similar to the following:
  ```shell
  2015-06-01 09:58:14 -0400 debug.test: {"json":"message"}
  ```
  If the fluent process is not running the response will be similar to the following:
  ```shell
  "connect failed: Connection refused - connect(2) for "127.0.0.1" port 24224"
  ```
  

    _**Note:** Typing `fluentd -help` gives an outline of `fluentd` commands available. If it throws error 'fluentd command not found', set `fluentd` binary path to PATH environment variable._

##References:
https://github.com/fluent/fluentd
