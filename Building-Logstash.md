<!---PACKAGE:Logstash--->
<!---DISTRO:RHEL 6.6:2.3.0--->
<!---DISTRO:RHEL 7.1:2.3.0--->
<!---DISTRO:SLES 11:2.3.0--->
<!---DISTRO:SLES 12:2.3.0--->
<!---DISTRO:Ubuntu 16.x:2.3.0--->

# Building Logstash
[Logstash](https://www.elastic.co/products/logstash) is written in Ruby and it has a build-in Jruby (running on JVM) that needs a native library jffi-1.2.so for s390x platform.

This recipe is for building Logstash (2.3.0) for Linux on z Systems (SLES12/SLES11/RHEL6/RHEL7/Ubuntu16.04)

_**General Notes:**_ 	 
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._


### 1. Dependencies:

 *	RHEL7:

	```
		sudo yum install -y java-1.7.0-openjdk ant make wget unzip tar gcc
	```
 *	RHEL6:
	```
		sudo yum install -y java-1.7.1-ibm-devel.s390x wget make unzip gcc tar
	```

 *	SLES12:
	```
		sudo zypper install -y --type pattern Basis-Devel
		sudo zypper install -y java-1_7_0-openjdk ant make wget unzip gcc
	```
	
 *	SLES11:
	```
        sudo zypper update
		sudo zypper install -y --type pattern Basis-Devel
        sudo zypper install -y java-1_7_0-ibm-devel wget unzip tar make gcc
	```

 *  Ubuntu 16.04
    ```
        sudo apt-get  install -y  ant make wget unzip tar gcc openjdk-8-jdk
    ```
	
 * Install appropriate version of Ant for SLES11 and RHEL6:
    ```
        export WORK_DIR=`pwd`
        wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.4-bin.tar.gz
        tar -zxvf apache-ant-1.9.4-bin.tar.gz
    ```
    
### 2. Build from Release: 

 1. Set Environment Variable
 	*  Set JAVA_HOME

    ```
        export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk (for rhel7)
        export JAVA_HOME=/usr/lib64/jvm/java (for sles12)
        export JAVA_HOME=/usr/lib64/jvm/java-1.7.0-ibm (for sles11)
        export JAVA_HOME=/usr/lib/jvm/java-1.7.1-ibm.s390x (for rhel6)
		export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x/jre (for Ubuntu)
    ```
    
 	* Set ANT PATH for RHEL6 and SLES11
    ```
        export PATH=$WORK_DIR/apache-ant-1.9.4/bin:$PATH
    ```
 2. Get Logstash release package and unzip it

   ```
       cd /<source_root>/
       wget https://download.elastic.co/logstash/logstash/logstash-2.3.0.zip
       unzip -u logstash-2.3.0.zip
   ```
 3. Jruby runs on JVM and needs a native library (libjffi-1.2.so: java foreign language interface). Get `jffi` source code and build with `ant`

	```
       cd /<source_root>/
       wget https://github.com/jnr/jffi/archive/master.zip
       mv master master.zip (Only for SLES11)
       unzip  master.zip 
       cd jffi-master
       ant
	```
	
 4.  Copy libjffi-1.2.so to Logstash folder
    ```
       cd /<source_root>/
       mkdir logstash-2.3.0/vendor/jruby/lib/jni/s390x-Linux
       cp jffi-master/build/jni/libjffi-1.2.so \
       logstash-2.3.0/vendor/jruby/lib/jni/s390x-Linux/libjffi-1.2.so
    ```

 5. Run Logstash
   ```
      cd /<source_root>/logstash-2.3.0
      bin/logstash version
   ```

## References:

* [logstash](https://www.elastic.co/products/logstash)