[Snappy-Java](https://github.com/xerial/snappy-java) is a library that provides Java bindings for the popular [snappy](http://google.github.io/snappy/) compression engine from Google. The [latest release in Maven Central](http://search.maven.org/#artifactdetails|org.xerial.snappy|snappy-java|1.1.2|bundle) already contains support for Linux on z Systems.

If you are running a Java application on Linux on z Systems that already bundles an older version of Snappy-Java, you can simply replace it with [the Snappy-Java 1.1.2 JAR file](http://search.maven.org/remotecontent?filepath=org/xerial/snappy/snappy-java/1.1.2/snappy-java-1.1.2.jar) from Maven Central. You may need to update the class path of the application, and potentially other meta-data used by the application to locate and load the JAR file.

## Building Snappy-Java from source

The following build instructions are not necessary if you only want to use pre-built Snappy-Java binaries (available from Maven Central; see above). The build instructions have been tested on RHEL 7, SLES 11 SP3 and SLES 12.

Currently it is not possible to build Snappy-Java correctly on RHEL 6, due to limitations in the libstdc++-static package. However, Snappy-Java can be built on RHEL 7, and the resulting JAR file will function correctly on a RHEL 6 system.

1. Install the prerequisites:

    (RHEL 7)

		yum install automake autoconf libtool pkgconfig gcc-c++ libstdc++-static \
                    java-1.7.1-ibm java-1.7.1-ibm-devel git wget tar make patch

    (SLES 11 SP3)

		zypper install automake autoconf libtool pkg-config gcc-c++ \
                    java-1_7_0-ibm java-1_7_0-ibm-devel git-core wget tar make patch

    (SLES 12)

		zypper install automake autoconf libtool pkg-config gcc-c++ \
                    java-1_7_1-ibm java-1_7_1-ibm-devel git-core wget tar make patch

   On RHEL 7, you will need the RHEL Optional and Supplementary repositories set up in order to install the libstdc++-static and IBM JDK packages. See this article for more information: [[Adding RHEL Optional and Supplementary Repositories]].

2. Check out the Snappy-Java source code:

        git clone https://github.com/xerial/snappy-java.git
        cd snappy-java
        git checkout develop

3. Set the JAVA_HOME environment variable to the Java 7 SDK directory, e.g.

        export JAVA_HOME=/etc/alternatives/java_sdk_ibm

   /etc/alternatives/java_sdk_ibm should be a symbolic link that points to the location where the IBM Java SDK is installed.

4. **(SLES 11 only)** Older versions of GCC do not support the -static-libstdc++ option, used in the Snappy-Java makefiles. To build on such a system, modify line 148 in Makefile.common so that it says:

        Linux-s390x_LINKFLAGS := -shared -static-libgcc -L .

   Then create a symbolic link to the static standard C++ library with this command:

        ln -s `g++ -print-file-name=libstdc++.a` .

   Doing this will allow the linker to find the static library first and use it instead of the dynamic libstdc++.so.

5. Issue this command to checkout the source code for Snappy, build the C++ code as well as the Java classes, and use Scala SBT to package the binaries into a JAR file:

        make IBM_JDK_7=1 USE_GIT=1 GIT_SNAPPY_BRANCH=master \
             GIT_REPO_URL=https://github.com/google/snappy.git test

   If this `make` command fails for any reason and you need to restart, issue `make clean` first. Otherwise the build will try to continue with an incomplete source tree and it will keep failing.

   This command will also run some unit tests to ensure that the JAR file works.

6. Issue this command to ensure that libsnappyjava.so is not linked against libstdc++.so and libgcc_s.so:

        ldd target/snappy-1.1.2-Linux-s390x/libsnappyjava.so

   You should see that only three dynamic libraries are linked:

        libm.so.6 => /lib64/libm.so.6 (0x000003fffd4b1000)
        libc.so.6 => /lib64/libc.so.6 (0x000003fffd2f6000)
        /lib/ld64.so.1 (0x000002aacc10d000)

7. The resulting JAR file can be found under snappy-java/target/ as snappy-java-1.1.2-SNAPSHOT.jar. It actually includes not only the native libraries for Linux on z Systems, but also for other platforms as well (e.g. Linux on x86, Windows, etc.).

8. Rename the JAR file to snappy-java-1.1.2.jar, and add it to the class path of any Java project that uses the snappy compression engine (e.g. Cassandra). It works as a drop-in replacement for any copy of Snappy-Java that comes with the Java project.
