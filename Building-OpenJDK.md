# Building OpenJDK9

The instructions provided below specify the steps to build OpenJDK9 on Linux on the IBM z Systems for RHEL 7.2.
Please note that OpenJDK9 is not GAed yet. Click [here](http://openjdk.java.net/projects/jdk9/) for details of release schedule. 

Please note that there are continuous updates that are happening on 'jdk9/hs' repository that may cause issues in build. One can subscribe to 's390x-port-dev@openjdk.java.net' mailing list to view/report issues.

_**General Notes:**_ 	
_1) When following the steps below please use a super user - root._  
_2) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._  

### Building OpenJDK
1. Install the build dependencies 

	For RHEL 7.2
	
	```
	yum install java-1.8.0-openjdk-devel file-devel unzip zip gcc make gcc-c++ libXtst-devel libXt-devel libXrender-devel libXi-devel cups-devel freetype-devel alsa-lib-devel tar which hg libffi-devel
	```

2. Download the source code for OpenJDK9
	
	```
	hg clone http://hg.openjdk.java.net/jdk9/hs openjdk9
	cd openjdk9
	bash ./get_source.sh
	```

3. Set the environment variables
	```
	unset JAVA_HOME
	export LANG=C
	export PATH="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.111-1.b15.el7_2.s390x/bin:${PATH}"
	```
    
4. Build OpenJDK9
   * JVM variant - Server (With JIT) _**[Please note that there is build issue with Server variant which is reported to s390x port developers]**_

	```
	cd <source_root>/openjdk9
	bash ./configure --with-jvm-variants=server --disable-warnings-as-errors
	make all
	```
   * JVM variant - Zero (Without JIT)

	```
	cd <source_root>/openjdk9
	bash ./configure --with-jvm-variants=zero --disable-warnings-as-errors
	make all
	```

4. Set Environment variables
 
	```
	export JAVA_HOME=<source_root>/openjdk9/build/linux-s390x-normal-server-release/jdk
	export PATH=$JAVA_HOME/bin:$PATH
	```

###References
 http://openjdk.java.net/    
http://hg.openjdk.java.net/jdk9/jdk9/raw-file/tip/README-builds.html   