The [Erlang/OTP 17.4](http://www.erlang.org/download_release/27) can be built on SLES 12 on IBM z System by following these instructions.

1. As root, install IBM JDK to build with the Java interface (jinterface):
        zypper in java-1_7_1-ibm java-1_7_1-ibm-devel

2. As root, install all the build-time dependencies:

        zypper in libopenssl-devel libssh-devel ncurses-devel unixODBC unixODBC-devel

3. Download the Erlang/OTP source code from Erlang's official website:

        wget http://www.erlang.org/download/otp_src_17.4.tar.gz

4. As a regular user, untar the source code and change working directory:

        tar zxvf otp_src_17.4.tar.gz
        cd otp_src_17.4
        export ERL_TOP=`pwd`

5. As a regular user, configure and make:

        ./configure --prefix=/usr
        make

6. To verify the build, start the Erlang shell:

        $ERL_TOP/bin/erl
    you should see a shell that looks like this:
        Erlang/OTP 17 [erts-6.3] [source] [64-bit] [smp:2:2] [async-threads:10] [kernel-poll:false]

        Eshell V6.3  (abort with ^G)
        1> 

7. Running the smoke tests:

        make release_tests
        cd release/tests/test_server
        $ERL_TOP/bin/erl -s ts install -s ts smoke_test batch -s init stop

8. **OPTIONAL** running the full functional verification test suites. This will take a long time (approximately 12 hours). In the Erlang shell (if configured with `--prefix=/usr` then start the shell at `/usr/bin/erl`):

        ts:install(). % install the ts framework
        ts:run(). % to run all the test suites

9. If no errors found in the smoke tests, as root, install Erlang:

        make install

You can now invoke the Erlang shell as `/usr/bin/erl` and EScript as `/usr/bin/escript`.
 