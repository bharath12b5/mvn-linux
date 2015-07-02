The [Erlang/OTP 17.4](http://www.erlang.org/download_release/27) can be built on SLES 11/12 and RHEL 6/7 IBM Linux on z Systems as follows :-

_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified._

1. Install the build time dependencies:

 (SLES 11)

    ```shell
   sudo zypper install java-1_7_0-ibm java-1_7_0-ibm-devel wget tar make perl gcc gcc-c++ libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel
    ```

   (SLES 12)

    ```shell
   sudo zypper install java-1_7_1-ibm java-1_7_1-ibm-devel wget tar make perl gcc gcc-c++ libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel
    ```

   (RHEL 6/7)
   
    ```shell
   sudo yum install java-1.7.1-ibm java-1.7.1-ibm-devel wget tar make perl gcc gcc-c++ openssl openssl-devel ncurses-devel ncurses unixODBC unixODBC-devel
    ```

2. Move to a working directory with write permission to build Erlang in.
 ```shell
 cd /<source_root>/
 ```

3. Download the Erlang/OTP source code from Erlang's official website:

 ```shell
 wget http://www.erlang.org/download/otp_src_17.4.tar.gz
 ```
 _**Note:** otp_src_17.4 was available at the time of writing, paths and filenames can vary, dependent on release._

4. Untar the source code and step down into a new working directory:

    ```shell
    tar zxf otp_src_17.4.tar.gz
    cd otp_src_17.4
    export ERL_TOP=$(pwd)
    ```
    _**Note:** The export variable ERL_TOP must be set exactly as shown to allow the tests to function correctly_

5. Configure and make Erlang:

    ```shell
    ./configure --prefix=/usr
    make
    ```

6. To verify the build, start the Erlang shell:

    ```shell
    $ERL_TOP/bin/erl
    ```

    You should see a shell as shown below:
    
    ```shell
    Erlang/OTP 17 [erts-6.3] [source] [64-bit] [smp:2:2] [async-threads:10] [kernel-poll:false]
    
    Eshell V6.3  (abort with ^G)
    1>
    ```
    _**Note:** To exit from the shell type `<ctrl>-G`,and then, once `User switch command` is displayed type `q` then return._

7. **OPTIONAL** Run the smoke tests:

    ```shell
    cd /<source_root>/otp_src_17.4 
    make release_tests
    cd release/tests/test_server
    $ERL_TOP/bin/erl -s ts install -s ts smoke_test batch -s init stop
    ```

8. **OPTIONAL** Run the full functional verification test suites. This will take a long time (approximately 12 hours):

    ```shell
    $ERL_TOP/bin/erl
    ```
    Then once in the Erlang shell enter
    ```erlang
    ts:install(). 
    ts:run().
    ```
    _**Note:** `ts:install().` installs the framework, and then `ts:run().` runs all the test suites._

10. If no error is found in the smoke tests, install Erlang:

    ```shell
    cd /<source_root>/otp_src_17.4 
    sudo make install
    ```

   You can now remove the `<source_root>` directory  and invoke the Erlang shell as `/usr/bin/erl` and EScript as `/usr/bin/escript`.