[Snappy-Java](https://github.com/xerial/snappy-java) is a library that provides Java bindings for the popular [snappy](http://code.google.com/p/snappy/) compression engine from Google. It has been patched to enable it to be built on Linux on z Systems. The following build instructions have been tested on RHEL 6, RHEL 7, SLES 11 SP3 and SLES 12.

1. Install the prerequisites:

    (RHEL 6 and RHEL 7)

		yum install automake autoconf libstdc++-static java-1.7.1-ibm java-1.7.1-ibm-devel git wget gcc-c++ libtool tar make patch pkgconfig

    (SLES 11 SP3)

		zypper install automake autoconf java-1_7_0-ibm java-1_7_0-ibm-devel git-core wget gcc-c++ libtool tar make patch pkg-config

    (SLES 12)

		zypper install automake autoconf java-1_7_1-ibm java-1_7_1-ibm-devel git-core wget gcc-c++ libtool tar make patch pkg-config

   If `yum` is unable to find the IBM Java packages on RHEL, you might not have the correct repositories set up. See this article for more information: [[Adding RHEL Optional and Supplementary Repositories]].

2. Check out the Snappy-Java source code:

        git clone https://github.com/xerial/snappy-java.git
        cd snappy-java
        git checkout develop

3. Set the JAVA_HOME environment variable to the Java 7 SDK directory, e.g.

        export JAVA_HOME=/etc/alternatives/java_sdk_ibm

   /etc/alternatives/java_sdk_ibm should be a symbolic link that points to the location where the IBM Java SDK is installed.

4. Issue this command to checkout the source code for Snappy, build the C++ code as well as the Java classes, and use Scala SBT to package the binaries into a JAR file:

        make IBM_JDK_7=1 USE_GIT=1 GIT_SNAPPY_BRANCH=master \
             GIT_REPO_URL=https://github.com/google/snappy.git

   If this `make` command fails for any reason and you need to restart, issue `make clean` first. Otherwise the build will try to continue with an incomplete source tree and it will keep failing.

5. The resulting JAR file can be found under snappy-java/target/ as snappy-java-1.1.2-SNAPSHOT.jar. It actually includes not only the native libraries for Linux on z Systems, but also for other platforms as well (e.g. Linux on x86, Windows, etc.).

6. Rename the JAR file to snappy-java-1.1.2.jar, and add it to the class path of any Java project that uses the snappy compression engine (e.g. Cassandra). It works as a drop-in replacement for any copy of Snappy-Java that comes with the Java project.