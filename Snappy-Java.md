<!---PACKAGE:Snappy-Java--->
<!---DISTRO:SLES 12:1.1.2--->
<!---DISTRO:SLES 11:1.1.2--->
<!---DISTRO:RHEL 7:1.1.2--->
<!---DISTRO:RHEL 6:1.1.2--->

# Building Snappy-Java

Below versions of Snappy-Java are available in respective distributions at the time of this recipe creation:

*    RHEL 7 & RHEL 6 has `1.1.0`
*    SLES 12 has `1.1.1-1.43`
*    Ubuntu 16.04 has `1.1.2.1-2`

[Snappy-Java](https://github.com/xerial/snappy-java) is a library that provides Java bindings for the popular [snappy](http://google.github.io/snappy/) compression engine from Google. The [latest release in Maven Central](http://search.maven.org/#artifactdetails|org.xerial.snappy|snappy-java|1.1.2.6|bundle) already contains support for Linux on z Systems.

If you are running a Java application on Linux on z Systems that already bundles an older version of Snappy-Java, you can simply replace it with [the Snappy-Java 1.1.2.6 JAR file](http://search.maven.org/remotecontent?filepath=org/xerial/snappy/snappy-java/1.1.2.6/snappy-java-1.1.2.6.jar) from Maven Central. You may need to update the class path of the application, and potentially other meta-data used by the application to locate and load the JAR file.

_**General Notes:**_ 

_When following the steps below please use a standard permission user unless otherwise specified._

## Building Snappy-Java from source

The following build instructions are not necessary if you only want to use pre-built Snappy-Java binaries (available from Maven Central; see above). The build instructions have been tested on RHEL 6, RHEL 7, SLES 11 SP3, SLES 12 and Ubuntu 16.04.

1. Install the prerequisites:

    RHEL 6/RHEL 7:
    ```
    sudo yum install automake which autoconf libtool pkgconfig gcc-c++ libstdc++-static java-1.7.1-ibm java-1.7.1-ibm-devel git wget tar make patch
    ```

    SLES 11 (SP3):
    ```
    sudo zypper install automake  autoconf libtool pkg-config gcc-c++ java-1_7_0-ibm java-1_7_0-ibm-devel-1.7.0_sr9.10-9.1.s390x git-core wget tar make patch
    ```

    SLES 12:
    ```
    sudo zypper install automake autoconf libtool pkg-config gcc-c++ java-1_7_1-ibm java-1_7_1-ibm-devel git-core wget tar make patch
    ```

    Ubuntu 16.04:
    ```    
    sudo apt-get update
    sudo apt-get install openjdk-8-jdk automake autoconf libtool pkg-config git wget tar make patch
    ```

   _**Note:**_ On RHEL 7, you will need the RHEL Optional and Supplementary repositories set up in order to install the libstdc++-static and IBM JDK packages. See this article for more information: [Adding RHEL Optional and Supplementary Repositories](https://github.com/linux-on-ibm-z/docs/wiki/Adding-RHEL-Optional-and-Supplementary-Repositories).

2. Check out the Snappy-Java source code:
    ```
    git clone https://github.com/xerial/snappy-java.git
    cd snappy-java
    git checkout 1.1.2.6
    ```

3. Set the JAVA_HOME environment variable
        
    RHEL/SLES:
    ```
    export JAVA_HOME=/etc/alternatives/java_sdk_ibm
    ```    
_**Note:**_ /etc/alternatives/java_sdk_ibm should be a symbolic link that points to the location where the IBM Java SDK is installed.

    Ubuntu 16.04:
    ```
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
    ```     

4. Issue this command to checkout the source code for Snappy, build the C++ code as well as the Java classes, and use Scala SBT to package the binaries into a JAR file
    ```
    make IBM_JDK_7=1 USE_GIT=1 GIT_SNAPPY_BRANCH=master GIT_REPO_URL=https://github.com/google/snappy.git 

    ``` 

    _**Note:**_ In case of `make` command fails for any reason, first apply `make clean` and then restart `make`. Otherwise the build will try to continue with an incomplete source tree and it will keep failing. 
This command will also run some unit tests to ensure that the JAR file works.     


5. Issue this command to ensure that libsnappyjava.so is not linked against libstdc++.so and libgcc_s.so:
    ```
    ldd target/snappy-1.1.2-Linux-s390x/libsnappyjava.so
    ```
    * Check dynamic library links as a results of above ldd command 

        ```
        libm.so.6 => /lib64/libm.so.6 (0x000003fffd009000)
        libc.so.6 => /lib64/libc.so.6 (0x000003fffce7b000)
        /lib/ld64.so.1 (0x000002aa000b5000)
        ```
    * In case of RHEL6 and SLES11 another two libraries will be linked  

        ```
        libstdc++.so.6 => /usr/lib64/libstdc++.so.6 (0x000003fffd4e6000)
        libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x000003fffd231000)
        ```        

    _**Note:**_ The resulting JAR file can be found under snappy-java/target/ as snappy-java-1.1.2.6.jar. It is used to add it to the class path of any Java project that uses the snappy compression engine (e.g. Cassandra). It works as a drop-in replacement for any copy of Snappy-Java that comes with the Java project.

6. Example to test Snappy-Java 1.1.2.6 library:

    Create Test.java in target folder, compile and run Test.java as shown below :

     ```shell
        import org.xerial.snappy.*;
        public class Test {
            public static void main(String[] args) {
                try {
                    String input = "Hello snappy-java! Snappy-java is a JNI-based wrapper of " + 
						"Snappy, a fast compresser/decompresser.";
					byte[] compressed = Snappy.compress(input.getBytes("UTF-8"));
					byte[] uncompressed = Snappy.uncompress(compressed);

					String result = new String(uncompressed, "UTF-8");
					System.out.println(result);
                } catch(Exception e){
                         System.out.println(e);
                }
            }
        }
    ```
    Compile java example: `javac -cp ".:snappy-java-1.1.2.6.jar" Test.java`

    Run example: `java -cp ".:snappy-java-1.1.2.6.jar" Test`

    Following ouput should be observed:

    ```Hello snappy-java! Snappy-java is a JNI-based wrapper of Snappy, a fast compresser/decompresser.```
    
    