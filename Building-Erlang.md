<!---PACKAGE:Erlang--->
<!---DISTRO:RHEL 6.6:19.x--->
<!---DISTRO:RHEL 7.1:19.x--->
<!---DISTRO:SLES 11:19.x--->
<!---DISTRO:SLES 12:19.x--->
<!---DISTRO:Ubuntu 16.x:Distro,19.x--->

# Building Erlang

Below versions of Erlang are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04/16.10 have `18.3`

The instructions provided below specify the steps to build Erlang version 19.2 on Linux on the IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.


_**General Notes:**_ 	 
  * _When following the steps below please use a standard permission user unless otherwise specified._

  * _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._ 

##Step 1: Building and Installing Erlang 
                  
####1.1) Install the build dependencies
  
  * RHEL 6.8 and RHEL 7.1/7.2/7.3
  ```
  sudo yum install java-1.7.1-ibm java-1.7.1-ibm-devel wget tar make perl gcc gcc-c++ openssl openssl-devel ncurses-devel ncurses unixODBC unixODBC-devel
  ```

  * SLES 11-SP4
  ```
  sudo zypper install java-1_7_0-ibm java-1_7_0-ibm-devel wget tar make perl gcc gcc-c++ libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel
  ```

  * SLES 12/12-SP1/12-SP2
  ```
  sudo zypper install java-1_7_1-ibm java-1_7_1-ibm-devel wget tar make perl gcc gcc-c++ libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel
  ```
  
  * Ubuntu 16.04/16.10
  ```
  sudo apt-get update
  sudo apt-get install wget tar make perl openssl gcc g++ libncurses-dev libncurses5-dev unixodbc unixodbc-dev libssl-dev openjdk-8-jdk
  ``` 
	  
####1.2) Move to a working directory with write permission to build Erlang
 
  ```
  cd /<source_root>/
  ```
####1.3) Download the Erlang/OTP source code from Erlang's official website
  ```
  wget http://www.erlang.org/download/otp_src_19.2.tar.gz
  ```

  _**Note:**_ *otp_src_19.2 was available at the time of writing, paths and filenames can vary, dependent on release.*

####1.4) Untar the source code and step down into a new working directory
   ```
  tar zxf otp_src_19.2.tar.gz
  cd otp_src_19.2
  export ERL_TOP=$(pwd)
   ```
 _**Note:**_ *The export variable ERL_TOP must be set exactly as shown to allow the tests to function correctly.*

####1.5) Modify the file `/<source_root>/otp_src_19.2/lib/os_mon/src/memsup.erl` as below
  ```diff
@@ -700,6 +700,7 @@
        "x86_64"  -> 64;
        "sparc64" -> 64;
        "amd64"   -> 64;
+       "s390x"   -> 64;
        "ppc64"   -> 64;
        _         -> 32
     end.
  ```
####1.6) Configure and make Erlang

  ```
  ./configure --prefix=/usr
  make
  ```
####1.7) To verify the build, start the Erlang shell
  ```
  $ERL_TOP/bin/erl
  ```
You should see a shell as shown below:

 	 > Erlang/OTP 19 [erts-8.2] [source] [64-bit] [smp:4:4] [async-threads:10] [kernel-poll:false]

 	 >Eshell V8.2  (abort with ^G)
 	
_**Note:**_ *To exit from the shell type \<ctrl\>-G, and then, once User switch command is displayed type q then return.*

####1.8) Install Erlang
  ```
  cd /<source_root>/otp_src_19.2 
  sudo make install
  ```

  You can now remove the <source_root> directory and invoke the Erlang shell as /usr/bin/erl and EScript as /usr/bin/escript.

##Step 2: Testing

####2.1) Run the smoke tests
  ```
  cd /<source_root>/otp_src_19.2 
  make release_tests
  cd release/tests/test_server
  $ERL_TOP/bin/erl -s ts install -s ts smoke_test batch -s init stop
  ```
_**Note:**_ *Based on your timezone, Smoke tests could fail 2 test cases in `time_SUITE` test suite. The test suite checks the output of `date +%Z`, and if it starts with "SAST", the test assumes that the time zone is UTC+2 all year; otherwise it assumes UTC+1 in the winter and UTC+2 in the summer (northern hemisphere). They can be fixed by editing File `time_SUITE.erl` at location `/<source_root>/otp_src_19.2/erts/emulator/test`.*

  For example: For timezone in EST following are the changes to be done in the file.
  
  ```diff
@@ -865,8 +865,8 @@
        case os:type() of
            {unix,_} ->
                case os:cmd("date '+%Z'") of
-                   "SAST"++_ ->
-                       {2,2};
+                   "EST"++_ ->
+                       {-5,-4};
                    _ ->
                        {?timezone,?dst_timezone}
                end;
  ```
  
####2.2) OPTIONAL - Run the full functional verification test suites. This will take a long time (approximately 12 hours)
 
  ```
  cd /<source_root>/otp_src_19.2 
  make release_tests
  cd release/tests/test_server
  $ERL_TOP/bin/erl
  ```
    
  Then once in the Erlang shell enter
    
  ```
  ts:install(). 
  ts:run().
  ```

  _**Note:**_ *ts:install(). installs the framework, and then ts:run(). runs all the test suites.*

##References:
https://www.erlang.org/
