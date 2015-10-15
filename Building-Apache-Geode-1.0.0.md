The [Apache Geode] (http://geode.incubator.apache.org/) can be built and tested on Linux on z Systems (RHEL 7.1 and SLES 11) by following these instructions.

Note: When following the steps below please use a standard permission user unless otherwise specified. It is suggested that you create a new working directory from which to execute the instructions below (unless otherwise specified).

1. Install Java JDK, for example IBM Java in RHEL:

    ```
    $ sudo yum install java-1.7.1-ibm-devel
    ```
   or Open JDK:

    ```
    $ sudo yum install java-1.8.0-openjdk
    ```

   Set up java environment:

    ```
    $ JAVA_HOME=/usr/lib/jvm/java-*.*.*/
    $ export JAVA_HOME
    ```

   If using SLES, simply replace `yum` with `zypper` in the above command to install packages.

2. Get the Geode source code:

    ```
    $ git clone https://github.com/apache/incubator-geode.git
    $ cd incubator-geode
    ```

3. Modify the build.gradle file in gemfire-core folder:

    ```
    $ vi gemfire-core/build.gradle
    ```

   In the dependencies section, change the snappy-java entry as follows, to allow the build to download the latest version of [[Snappy-Java]] from Maven Central:

    ```
    dependencies{
        ...
        compile 'org.xerial.snappy:snappy-java:1.1.2'
    }
    ```

4. **Note: This step is needed only when using IBM Java. When using OpenJDK, skip to the next step.**

   Patch the source code to make it work correctly with IBM Java:

    ```
    $ vi gemfire-core/src/main/java/com/gemstone/gemfire/internal/cache/xmlcache/CacheXmlParser.java
    ```

   Navigate to function `public static CacheXmlParser parse(InputStream is)`, and add the following code at the beginning of this function,

    ```java
    class NeverCloseInputStream extends BufferedInputStream
    {
        public NeverCloseInputStream(InputStream stream)
        {
            super(stream);
        }

        @Override
        public void close()
        {
        }
    }
    ```

   And also change the line `BufferedInputStream bis = new BufferedInputStream(is);` to

    ```
    NeverCloseInputStream bis = new NeverCloseInputStream(is);
    ```

   Then save the file.

5. Build and install. You should see the successful build and test results.

    ```
    $ ./gradlew build installDist
    ```

### References

1. [Downloading Apache Geode](http://geode.incubator.apache.org/download/)
2. [[Snappy-Java]]
