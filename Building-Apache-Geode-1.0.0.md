The [Apache Geode] (http://geode.incubator.apache.org/) can be built and tested on Linux on z System (RHEL 7.1 ans SLES 11) by following these instructions.

NOTE: When following the steps below please use a standard permission user unless otherwise specified. It is suggested that you create a new working directory from which to execute the instructions below (unless otherwise specified).

1. Install Java JDK, for example IBM Java in RHEL

    ```
    $ sudo yum install java-1.7.1-ibm-devel
    ```
   or Open JDK

    ```
    $ sudo yum install java-1.8.0-openjdk
    ```

   Set up java environment:

    ```
    $ JAVA_HOME=/usr/lib/jvm/java-*.*.*/
    $ export JAVA_HOME
    ```

    If using SLES, simply replace `yum` with `zypper` in the above command to install packages.

2. Get Geode package

    ```
    $ git clone https://github.com/apache/incubator-geode.git
    $ cd incubator-geode
    ```

3. Build a Snappy-java jar, which contains s390x support, following https://github.com/linux-on-ibm-z/docs/wiki/Building-Snappy-Java. Put the generated jar file, i.e., snappy-java-1.1.2.jar, in a directory, for example DIR.

4. Change the build.gradle file in gemfire-core folder, to include the new Snappy-java.jar

    ```
    $ vi gemfire-core/build.gradle
    ```

    In dependency section, change the snappy-java entry with following changes

    ```
    dependencies {
    ....
        //compile 'org.xerial.snappy:snappy-java:1.1.2-RC3'
        compile files('$DIR/snappy-java-1.1.2.jar')
    }
    ```

5. **Note: Step 5 is needed when using IBM Java only. If using openJDK, skip to step 6 directly.**
   Modify source code to support IBM JAVA, assuming you are now in the geode directory

    ```
    $ vi ./gemfire-core/src/main/java/com/gemstone/gemfire/internal/cache/xmlcache/CacheXmlParser.java
    ```

   Navigate to function `public static CacheXmlParser parse(InputStream is)`, and add the following code at the beginning of this function,

    ```
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

6. Build and install. You should see the successful build and tests get passed.

    ```
    $ ./gradlew build installDist
    ```

### Note:
In the recipe, users are required to build their snappy-java themselves. This is because the official up-to-date snappy-java:1.1.2-RC3 (up to June 29, 2015) doesn't contain s390x support, but we are expecting this to happen very soon since the support was already up-streamed.

Once the new version of snappy-java (with s390x support, saying snappy-java-1.1.2-RCx.jar) is released and published, we recommend the users to import snappy-java with the following approach:

Modify the dependency section in `$DIR/gemfire-core/build.gradle` by updating the snappy-java entry:

    dependencies{
        ...
        compile 'org.xerial.snappy:snappy-java:1.1.2-RCx'
    }

### Reference
1. Download Geode: http://geode.incubator.apache.org/download/
2. Snappy-java recipe: https://github.com/linux-on-ibm-z/docs/wiki/Building-Snappy-Java