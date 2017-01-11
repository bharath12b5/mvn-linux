<!---PACKAGE:Logstash--->
<!---DISTRO:RHEL 6.x:5.1--->
<!---DISTRO:RHEL 7.X:5.1--->
<!---DISTRO:SLES 11.x:5.1--->
<!---DISTRO:SLES 12.x:5.1--->
<!---DISTRO:Ubuntu 16.x:5.1--->


# Building Logstash
[Logstash](https://www.elastic.co/products/logstash) is written in Ruby and it has a build-in Jruby (running on JVM) that needs a native library jffi-1.2.so for s390x platform.

The instructions provided below specify the steps to build Apache Logstash v5.1.1 on the IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	 
  * _When following the steps below please use a standard permission user unless otherwise specified._

  * _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._


## Step 1: Building and installing Logstash  
####1.1) Install dependencies


 *	RHEL 6.8
	```
		sudo yum install -y java-1.8.0-ibm.s390x java-1.8.0-ibm-devel.s390x wget unzip tar gcc make
	```

 *	RHEL 7.1/7.2/7.3  

	With IBM JDK:

	```
		sudo yum install -y java-1.8.0-ibm-devel.s390x ant wget unzip make gcc
	```  

	With OpenJDK:  
	```
		sudo yum install -y java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x ant wget unzip make gcc
	```

 *	SLES 11-SP4  
	```
	   sudo zypper install -y wget tar gcc make
	```

	* To install IBM Java 8, download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.

 *	SLES 12  
	```
	   sudo zypper install -y ant wget unzip make gcc
	```

	* To install IBM Java 8, download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.

 *	SLES 12-SP1/12-SP2

	With IBM JDK:

	```
		sudo zypper install -y --type pattern Basis-Devel
		sudo zypper install -y java-1_8_0-ibm-devel ant wget unzip make gcc
	```

	With OpenJDK:  
	```
		sudo zypper install -y java-1_8_0-openjdk java-1_8_0-openjdk-devel ant wget unzip make gcc
	```

 *  Ubuntu 16.04/16.10 

	With IBM JDK:
			
			sudo apt-get install -y ant make wget unzip tar gcc 
			
	* Install IBM Java 8, download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.  

	With OpenJDK:
		
			sudo apt-get install -y ant make wget unzip tar gcc openjdk-8-jdk

	
 * Install other dependency for RHEL 6.8 and SLES 11-SP4 i.e. **Ant**:
    ```bash
        export WORK_DIR=`pwd`
        wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.4-bin.tar.gz
        tar -zxvf apache-ant-1.9.4-bin.tar.gz
    ```
    
####1.2) Set Environment Variable
      
```bash  
      export JAVA_HOME=<path to java>
      export PATH=$JAVA_HOME/bin:$PATH
      export PATH=$WORK_DIR/apache-ant-1.9.4/bin:$PATH(for RHEL6.8 and SLES11-SP4)  
```  

####1.3) Download source code of Logstash

```bash
      cd /<source_root>/
      wget https://artifacts.elastic.co/downloads/logstash/logstash-5.1.1.zip
      unzip -u logstash-5.1.1.zip
```

 Jruby runs on JVM and needs a native library (libjffi-1.2.so: java foreign language interface). Get `jffi` source code and build with `ant`
```bash
       cd /<source_root>/
       wget https://github.com/jnr/jffi/archive/master.zip
       mv master master.zip (Only for SLES11-SP4)
       unzip  master.zip 
       cd jffi-master
       ant
```
####1.4) Copy libjffi-1.2.so to Logstash folder
```bash
       cd /<source_root>/
       mkdir logstash-5.1.1/vendor/jruby/lib/jni/s390x-Linux
       cp jffi-master/build/jni/libjffi-1.2.so \
       logstash-5.1.1/vendor/jruby/lib/jni/s390x-Linux/libjffi-1.2.so
```
##Step 2: Run Logstash
```bash
      cd /<source_root>/logstash-5.1.1
      bin/logstash -V
```

## References:

* [logstash](https://www.elastic.co/products/logstash)
