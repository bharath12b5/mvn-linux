[Logstash](https://www.elastic.co/products/logstash) is written in Ruby and it has a build-in Jruby (running on JVM) that needs a native library `jffi-1.2.so` for s390x platform

This recipe is for building Logstash (1.5.3) for Linux on z Systems (SLES12/SLES11/RHEL6/RHEL7)

### Dependencies:
   - Java JDK 7 (or later, OpenJDK or IBM JDK)      
   - Ant

### Build from Release

Download [Logstash](https://www.elastic.co/products/logstash) package and place the `libjffi-1.2.so` for s390x platform

1. Install `ant` as root (skip if it is already installed)

        $ sudo yum install ant.noarch           # for RHEL or
        $ sudo zypper install ant               # for SLES


2. Get Logstash release package and unzip it

        $ wget https://download.elastic.co/logstash/logstash/logstash-1.5.3.zip
        $ unzip logstash-1.5.3.zip

    If  `wget` is not installed, perform

        $ yum install wget              # for RHEL or
        $ zypper install wget           # for SLES

3. Jruby runs on JVM and needs a native library (`libjffi-1.2.so`: java foreign language interface). Get `jffi` source code and build with `ant`

        $ wget https://github.com/jnr/jffi/archive/master.zip
        $ unzip  master.zip      
        $ cd jffi-master
        $ ant                   #  build libjffi-1.2.so from c

    `libjffi-1.2.so` is generated in folder: `jffi-master/build/jni/libjffi-1.2.so`

4. Copy `libjffi-1.2.so` to Logstash folder

        $ mkdir logstash-1.5.3/vendor/jruby/lib/jni/s390x-Linux
        $ cp jffi-master/build/jni/libjffi-1.2.so logstash-1.5.3/vendor/jruby/lib/jni/s390x-Linux/libjffi-1.2.so

5. Since version 1.5.2 the Logstash package introduces a ruby function to check user environment JRE version, it gives warning message if the JRE version is too low, however one of  ruby method "captures" that needs JRE seems not work with  IBM JRE 7 & 8.  ( No problem found when using Open JRE 7 or later). As a workaround, in file:

        logstash-1.5.3/vendor/bundle/jruby/1.9/gems/logstash-core-1.5.3-java/lib/logstash/runner.rb

    comment out `line 32`,  like:

        #    LogStash::Util::JavaVersion.warn_on_bad_java_version

6. Run Logstash

        $ cd  logstash-1.5.3
        $ bin/logstash version

Logstash should run on all supported OSes: SESL12/SESL11/RHEL6/RHEL7
