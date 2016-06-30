<!---PACKAGE:Erlang--->
<!---DISTRO:RHEL 6.6:18.3--->
<!---DISTRO:RHEL 7.1:18.3--->
<!---DISTRO:SLES 11:18.3--->
<!---DISTRO:SLES 12:18.3--->
<!---DISTRO:Ubuntu 16.x:Distro--->

# Building Erlang

Below versions of Erlang are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `18.3`

The instructions provided below specify the steps to build Erlang version 18.3 on Linux on the IBM z Systems for RHEL 6.6/7.1, SLES 11/12.


_**General Notes:**_ 	 
_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._ 

 1. Install the build dependencies:

      (SLES 11)
     ```
    sudo zypper install java-1_7_0-ibm java-1_7_0-ibm-devel wget tar make perl gcc gcc-c++ libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel
     ```

      (SLES 12)
     ```
     sudo zypper install java-1_7_1-ibm java-1_7_1-ibm-devel wget tar make perl gcc gcc-c++ libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel
    ```

       (RHEL 6.6/7.1)
      ```
     sudo yum install java-1.7.1-ibm java-1.7.1-ibm-devel wget tar make perl gcc gcc-c++ openssl openssl-devel ncurses-devel ncurses unixODBC unixODBC-devel
      ```
 2. Move to a working directory with write permission to build Erlang in:
 
      ```
      cd /<source_root>/
      ```
 3. Download the Erlang/OTP source code from Erlang's official website:
     ```
     wget http://www.erlang.org/download/otp_src_18.3.tar.gz
     ```

 **Note:** *otp_src_18.3 was available at the time of writing, paths and filenames can vary, dependent on release.*

 4. Untar the source code and step down into a new working directory:
     ```
     tar zxf otp_src_18.3.tar.gz
     cd otp_src_18.3
     export ERL_TOP=$(pwd)
     ```
 **Note:** *The export variable ERL_TOP must be set exactly as shown to allow the tests to function correctly.*

 5. Configure and make Erlang:
     ```
     ./configure --prefix=/usr
     make
     ```
 6. To verify the build, start the Erlang shell:
     ```
     $ERL_TOP/bin/erl
     ```
	You should see a shell as shown below:

 	 > Erlang/OTP 18 [erts-7.3] [source] [64-bit] [smp:4:4] [async-threads:10] [kernel-poll:false]

 	 >Eshell V7.3  (abort with ^G)
 	
  	**Note:** *To exit from the shell type \<ctrl\>-G, and then, once User switch command is displayed type q then return.*

 7. OPTIONAL Run the smoke tests:
    ```
     cd /<source_root>/otp_src_18.3 
     make release_tests
     cd release/tests/test_server
     $ERL_TOP/bin/erl -s ts install -s ts smoke_test batch -s init stop
     ```
    
 8. OPTIONAL Run the full functional verification test suites. This will take a long time (approximately 12 hours):
 
    ```
    $ERL_TOP/bin/erl
    ```
    
    Then once in the Erlang shell enter
    
    ```
    ts:install(). 
    ts:run().
    ```
    
 	 **Note:** *ts:install(). installs the framework, and then ts:run(). runs all the test suites.*

 9. If no error is found in the smoke tests, install Erlang:
     ```
     cd /<source_root>/otp_src_18.3 
     sudo make install
     ```
     
   	You can now remove the <source_root> directory and invoke the Erlang shell as /usr/bin/erl and EScript as /usr/bin/escript.