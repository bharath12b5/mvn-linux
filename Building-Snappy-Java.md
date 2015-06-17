[Snappy-Java](https://github.com/xerial/snappy-java) is a library that provides Java bindings for the popular [snappy](http://code.google.com/p/snappy/) compression engine from Google. It has been patched to enable it to be built on Linux on z Systems. The following build instructions have been tested on SLES 11 SP3.

1. Install the prerequisites:

		zypper install automake autoconf java-1_7_0-ibm java-1_7_0-ibm-devel git-core

2. Check out the Snappy-Java source code:

        git clone https://github.com/linux-on-ibm-z/snappy-java.git
        cd snappy-java
        git checkout develop-s390x

3. Set the JAVA_HOME environment variable to the IBM Java 7 directory, e.g.

        export JAVA_HOME=/usr/lib64/jvm/java-1.7.0-ibm-1.7.0

4. Issue this command to checkout the source code for Snappy, build the C++ code as well as the Java classes, and use Scala SBT to package the binaries into a JAR file:

        make IBM_JDK_7=1 USE_GIT=1 GIT_SNAPPY_BRANCH=master \
             GIT_REPO_URL=https://github.com/google/snappy.git

5. The resulting JAR file can be found under snappy-java/target/ as snappy-java-1.1.2-SNAPSHOT.jar. It actually includes not only the native libraries for z Systems, but also for other architecture as well (e.g. Linux on x86, Windows, etc.).

6. Rename the JAR file to snappy-java-1.1.2.jar, and add it to the class path of any Java project that uses the snappy compression engine (e.g. Cassandra). It works as a drop-in replacement for any copy of Snappy-Java that comes with the Java project.