# Building Erlang

The [Erlang/OTP 18.2.1](http://www.erlang.org/download_release/30) can be built on SLES 11/12 and RHEL 6.6/7.1 IBM Linux on z Systems as follows :-

_**General Notes:**_ 	 
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._ 

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
     wget http://www.erlang.org/download/otp_src_18.2.1.tar.gz
     ```

 **Note:** *otp_src_18.2.1 was available at the time of writing, paths and filenames can vary, dependent on release.*

 4. Untar the source code and step down into a new working directory:
     ```
     tar zxf otp_src_18.2.1.tar.gz
     cd otp_src_18.2.1
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

 	 > Erlang/OTP 18 [erts-7.2.1] [source] [64-bit] [smp:4:4] [async-threads:10] [kernel-poll:false]

 	 >Eshell V7.2.1  (abort with ^G)
 	
  	**Note:** *To exit from the shell type \<ctrl\>-G,and then, once User switch command is 	displayed type q then return.*

 7. OPTIONAL Run the smoke tests:
    ```
     cd /<source_root>/otp_src_18.2.1 
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
     cd /<source_root>/otp_src_18.2.1 
     sudo make install
     ```
     
   	You can now remove the <source_root> directory and invoke the Erlang shell as /usr/bin/erl and EScript as /usr/bin/escript.