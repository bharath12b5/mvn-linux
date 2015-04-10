The [Erlang/OTP 17.4](http://www.erlang.org/download_release/27) can be built on RHEL 7 on IBM z System by following these instructions.

1. As root, add the RHEL 7 Server Optional repository to the yum repository list. This is required to get the IBM Java package. See the instructions in [[Adding RHEL Optional and Supplementary Repositories]].

2. As root, install IBM JDK to build with the Java interface (jinterface):

        yum install java-1.7.1-ibm

3. As root, install all the build-time dependencies:

        yum install openssl openssl-devel ncurses-devel ncurses unixODBC unixODBC-devel

4. Download the Erlang/OTP source code from Erlang's official website:

        wget http://www.erlang.org/download/otp_src_17.4.tar.gz

5. As a regular user, untar the source code and change working directory:

        tar zxvf otp_src_17.4.tar.gz
        cd otp_src_17.4
        export ERL_TOP=`pwd`

6. As a regular user, configure and make:

        ./configure --prefix=/usr
        make

7. To verify the build, start the Erlang shell:

        $ERL_TOP/bin/erl

   You should see a shell that looks like this:

        Erlang/OTP 17 [erts-6.3] [source] [64-bit] [smp:2:2] [async-threads:10] [kernel-poll:false]
        
        Eshell V6.3  (abort with ˆG)
        1>

8. Running the smoke tests:

        make release_tests
        cd release/tests/test_server
        $ERL_TOP/bin/erl -s ts install -s ts smoke_test batch -s init stop

9. **OPTIONAL** Run the full functional verification test suites. This will take a long time (approximately 12 hours). In the Erlang shell (if configured with `--prefix=/usr` then start the shell with `/usr/bin/erl`):

        ts:install(). % install the ts framework
        ts:run(). % to run all the test suites

10. If no error is found in the smoke tests, install Erlang as root:

        make install

   You can now invoke the Erlang shell as `/usr/bin/erl` and EScript as `/usr/bin/escript`.