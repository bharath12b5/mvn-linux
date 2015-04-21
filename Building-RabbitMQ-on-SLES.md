The following build instructions have been tested with **RabbitMQ Server 3.5.0** on **SLES 11.3, 12.0 on IBM z Systems**.

Installation instructions are available in the [official RabbitMQ documentation](https://www.rabbitmq.com/build-server.html).

### Install the Dependencies

_Required build/runtime dependencies:_

1. Erlang > R13. See this article for complete build/install instructions: [Building Erlang on z](https://github.com/linux-on-ibm-z/docs/wiki/Building-Erlang-on-SLES12).

2. libxslt

3. Python >= 2.6.0 (available on vanilla SLES 11.3, 12.0).

4. Openssl (available on vanilla SLES 11.3, 12.0).

        sudo zypper in libxslt

_Optional Dependencies:_

1. xmlto (only required for building documentation).

2. Git (optional - only required if testing RabbitMQ Server using the umbrella).

3. Subversion (optional - only required if testing RabbitMQ Server using the umbrella).

4. Java > 1.5 (optional - only required if testing RabbitMQ Server using the Java client).

5. Ant 1.9 (optional - only required if testing RabbitMQ Server using the Java client).

6. Junit 3.8.1 (optional - only required if testing RabbitMQ Server using the Java client).

        # installation steps:
        sudo zypper in xmlto git subversion java-1_7_0-ibm-devel ant ant-junit


The server can be build from a standalone tarball. Alternatively the [RabbitMQ umbrella](https://www.rabbitmq.com/plugin-development.html) can be used for for building the server. Using the umbrella is recommended if you intend to dabble in the RabbitMQ source or develop plugins for Rabbit. The sections below will cover building and installing the RabbitMQ Server 3.5.0 using both of the options.

### Build, Test and Install the RabbitMQ Server 3.5.0

1. Download the source.

        wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.5.0/rabbitmq-server-3.5.0.tar.gz
        tar xvzf rabbitmq-server-3.5.0.tar.gz
        cd rabbitmq-server-3.5.0

2. The `all` target in the Makefile will build and test the server. A non-zero make return value indicates a failure. Unit test results will be output to the console.

        make all

3. Install the server, by setting the location of the RabbitMQ server binary, command and documentation respectively in the variables - `MAN_DIR, TARGET_DIR, SBIN_DIR`.

        sudo TARGET_DIR=<dir> SBIN_DIR=<dir> MAN_DIR=<dir> make install

    For example:

        sudo TARGET_DIR=/usr/bin SBIN_DIR=/usr/sbin MAN_DIR=/usr/share/man make install

<a name="build" />
### Build, test and Install the RabbitMQ Server 3.5.0 under the RabbitMQ Umbrella

1. Download the umbrella.

        git clone https://github.com/rabbitmq/rabbitmq-public-umbrella.git
        cd rabbitmq-public-umbrella

2. Download the RabbitMQ Server 3.5.0

        wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.5.0/rabbitmq-server-3.5.0.tar.gz

3. Build, test and install the server as outlined under [Build, Test and Install the RabbitMQ Server 3.5.0](#build) .

        mv rabbitmq-server-3.5.0 rabbitmq-server # needed for the macro's used in the umbrella Makefile
        cd rabbitmq-server
        make all
        sudo TARGET_DIR=<dir> SBIN_DIR=<dir> MAN_DIR=<dir> make install

4. (Optional) Test the RabbitMQ server with the Erlang client 3.5.0 test suite, Java client 3.5.0 test suite and Python QPid test suite.
First, download the relevant packages for the 3.5.0 distro. The Makefile in the umbrella contains targets for downloading additional RabbitMQ repositories. Refer to the Makefile to download repositories as per your need.

        cd ..

        # download the clients for version 3.5.0.
        wget https://www.rabbitmq.com/releases/rabbitmq-java-client/v3.5.0/rabbitmq-java-client-3.5.0.tar.gz
        wget https://www.rabbitmq.com/releases/rabbitmq-erlang-client/v3.5.0/amqp_client-3.5.0-src.tar.gz

        # extract package and rename, so they can be used by other make targets.
        tar -xvf rabbitmq-java-client-3.5.0.tar.gz
        mv rabbitmq-java-client-3.5.0 rabbitmq-java-client

        tar -xvf amqp_client-3.5.0-src.tar.gz
        mv amqp_client-3.5.0 rabbitmq-erlang-client

        # download the other repositories for testing rabbit.
        make checkout rabbitmq-test

        # run all of the test suites.
        # The console output contains test results, ../rabbitmq-java-client/build contains the Java test results.
        # On success, make returns with 0 status, any other status indicated a test failure.
        cd rabbitmq-test
        make all

### Notes on Verification Test Failures (not specific to Linux on z Systems)

On the machines running the IBM JVM, the Java Client SSL test suite fails with the test source as it is.

**Failure**
The Java test suite seems to be failing on the SSL tests and in particular with the following exception.
>java.security.NoSuchAlgorithmException: SSLv3 SSLContext not available

**Affected Source Files**

1. rabbitmq-java-client/test/src/com/rabbitmq/client/test/ssl/VerifiedConnection.java
2. rabbitmq-java-client/test/src/com/rabbitmq/client/test/ssl/BadVerifiedConnection.java

**Reason**
SunX509 is not availble on the IBM JVM along with SSLv3. In particular,

    TrustManagerFactory tmf = TrustManagerFactory.getInstance("SunX509");
    SSLContext c = SSLContext.getInstance("SSLv3");

**Fix**
The following patch should pass on any JVM since the default algorithm is JVM dependant. Moreover, SSLv3 is deprecated and should never be used.

    TrustManagerFactory tmf = TrustManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
    SSLContext c = SSLContext.getInstance("SSL");


### References

1. [Official Documentation](https://www.rabbitmq.com/build-server.html)
2. [Official Community Forum](https://groups.google.com/forum/#!topic/rabbitmq-users/)
3. [Discussion regarding the SSL test suite failure](http://www-01.ibm.com/support/knowledgecenter/SSYKE2_7.0.0/com.ibm.java.security.component.71.doc/security-component/jsse2Docs/sslcontext.html
http://cr.openjdk.java.net/˜xuelei/7093640/webrev.01/raw_files/new/test/sun/security/ssl/com/sun/net/ssl/internal/ssl/SSLContextImpl/DefaultEnabledProtocols.Java)
4. [Community Question regarding test failures](https://groups.google.com/forum/#!topic/rabbitmq-users/R7jTAm8z7dQ)