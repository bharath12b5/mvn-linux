<!---PACKAGE:Fluentd--->
<!---DISTRO:SLES 12:0.14--->
<!---DISTRO:SLES 11:0.14--->
<!---DISTRO:RHEL 7.1:0.14--->
<!---DISTRO:RHEL 6.6:0.14--->
<!---DISTRO:Ubuntu 16.x:0.14--->

# Building Fluentd

The instructions provided below specify the steps to build Fluentd 0.14.9 on IBM z Systems for RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._  

1. Install the dependencies

	Ubuntu 16.04/16.10
	```
	sudo apt-get install -y ruby ruby-dev gcc make
	```
2. For platforms RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2 build Ruby 2.3.1 from the instructions [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby)

    Once the Ruby build is completed and installed continue with the instructions below

3. Setup the gem HOME and PATH environment for a standard user

    ```shell
    export GEM_HOME=/home/<USER>/.gem/ruby
    export PATH=/home/<USER>/.gem/ruby/bin:$PATH
    ```  
    Where `<USER>` is the standard user you are logged in as
	
4. Build and install the latest stable fluentd gem and dependencies

    Fluentd is a gem based product, so the rubygem install process will automatically build the necessary native parts
	
    ```shell
    gem install fluentd -v 0.14.9
    ```
    Once complete it should report that a number of gems including fluentd are installed. Verify the installed version with `gem list fluentd`

5. Quick test of the fluentd installation(Optional)

	This test is to setup/install a config directory, then start a fluent process which is put into the background. Finally a message is piped to `fluent-cat`

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
