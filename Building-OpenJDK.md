# Building OpenJDK9

The instructions provided below specify the steps to build OpenJDK 9 on Linux on the IBM z Systems for RHEL 7.2.
Please note that OpenJDK9 is not GAed yet. Click [here](http://openjdk.java.net/projects/jdk9/) for details of realease schedule.

_**General Notes:**_ 	
_i) When following the steps below please use a super user - root._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._  

### Building OpenJDK
1. Install the build dependencies 

	For RHEL 7.2
	
	```
	yum install java-1.8.0-openjdk-devel file-devel unzip zip gcc make gcc-c++ libXtst-devel libXt-devel libXrender-devel libXi-devel cups-devel freetype-devel alsa-lib-devel tar which hg
	```

2. Download the source code for OpenJDK9
	
	```
	hg clone http://hg.openjdk.java.net/jdk9/hs openjdk9
	cd openjdk9
	sh get_source.sh
	```

3. Build OpenJDK9 (with JIT)

	```
	cd <source_root>/openjdk9
	bash configure --disable-hotspot-gtest --disable-warnings-as-errors
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
