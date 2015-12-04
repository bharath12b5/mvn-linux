# Building Logstash
[Logstash](https://www.elastic.co/products/logstash) is written in Ruby and it has a built-in Jruby (running on JVM) that needs a native library jffi-1.2.so for s390x platform.

This recipe is for building Logstash (2.1.0) for Linux on z Systems (SLES12/SLES11/RHEL6/RHEL7)

_**General Notes:**_ 	 
_When following the steps below please use a standard permission user unless otherwise specified._


### 1. Dependencies:

 *	RHEL7:

	```
		yum install -y java-1.7.0-openjdk ant make wget unzip tar gcc
	```
 *	RHEL6:
	```
		yum install -y java-1.7.1-ibm-devel.s390x wget make unzip gcc tar
	```

 *	SLES12:
	```
		zypper install -y --type pattern Basis-Devel
		zypper install -y java-1_7_0-openjdk ant make wget unzip gcc
	```
	
 *	SLES11:
	```
		zypper install -y --type pattern Basis-Devel
        zypper install -y java-1_7_0-ibm-devel wget unzip tar make gcc
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
    ```
    
 	* Set ANT PATH for RHEL6 and SLES11
    ```
        export PATH=$WORK_DIR/apache-ant-1.9.4/bin:$PATH
    ```
 2. Get Logstash release package and unzip it

   ```
       wget https://download.elastic.co/logstash/logstash/logstash-2.1.0.zip
       unzip -u logstash-2.1.0.zip
   ```
 3. Jruby runs on JVM and needs a native library (libjffi-1.2.so: java foreign language interface). Get `jffi` source code and build it with `ant`

	```
       wget https://github.com/jnr/jffi/archive/master.zip
       mv master master.zip (Only for SLES11)
       unzip  master.zip 
       cd jffi-master
       ant
	```
	
 4.  Copy libjffi-1.2.so to Logstash folder
    ```
       mkdir logstash-2.1.0/vendor/jruby/lib/jni/s390x-Linux
       cp jffi-master/build/jni/libjffi-1.2.so \
       logstash-2.1.0/vendor/jruby/lib/jni/s390x Linux/libjffi-1.2.so
    ```

 5. Run Logstash
   ```
      cd logstash-2.1.0
      bin/logstash version
   ```

## References:

* [logstash](https://www.elastic.co/products/logstash)
