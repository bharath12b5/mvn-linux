The [Apache Geode] (http://geode.incubator.apache.org/) can be built and tested on Linux on z System (RHEL 6,7 and SLES 11, 12) by following these instructions.

NOTE: When following the steps below please use a standard permission user unless otherwise specified. It is suggested that you create a new working directory from which to execute the instructions below (unless otherwise specified).

1. Install the IBM JDK 1.7.1 or 1.8.0; refer to [[Adding RHEL Optional and Supplementary Repositories]] https://github.com/linux-on-ibm-z/docs/wiki/Adding-RHEL-Optional-and-Supplementary-Repositories for detailed instructions.

    ```
    $ sudo yum install java-1.7.1-ibm-devel
    ```
   or Open JDK

    ```
    $ sudo yum install java-1.8.0-openjdk
    ```

2. Set up Java environment:

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

3. Change gemfire-core/build.gradle to include snappy-java-1.1.2.jar:

    ```
    $ vi gemfire-core/build.gradle
    ```

    In the dependencies section, change the snappy-java entry as follows:

    ```
    dependencies {
    ....
        compile 'org.xerial.snappy:snappy-java:1.1.2'
    }
    ```

4. Build and install. The build should succeed and tests should pass.

    ```
    $ ./gradlew build installDist
    ```

### References
1. Download Geode: http://geode.incubator.apache.org/download/
2. Snappy-Java recipe: https://github.com/linux-on-ibm-z/docs/wiki/Building-Snappy-Java