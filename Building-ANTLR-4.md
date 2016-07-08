## Building AntLR 4.5.1
Below versions of AntLR are available in respective distributions at the time of this recipe creation:

  * RHEL6 has `2.7.7`  
  * Ubuntu 16.04 has `4.5.1`

The instructions provided below specify the steps to build [AntLR](http://www.antlr.org/) version 4.5.1 on Linux on the IBM z Systems for RHEL 6/7 & SLES 11/12.

#### _**General Notes:**_
i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

## Building ANTLR

### Obtain pre-built dependencies 
Use the following commands to obtain dependencies
 
 RHEL 7.1
 
` yum install -y wget git java-1.7.1-ibm-devel ant`

 RHEL 6.6
	
` yum install -y wget git java-1.7.1-ibm-devel`

 SLES 12

` zypper install -y wget git-core java-1.7.1-ibm-devel ant`

 SLES 11

` zypper install -y wget git-core java-1_7_0-ibm-devel `
	   
	   
### Dependency build - Maven 3.3.9
This recipe uses Maven (version >= 3.3.3) to build ANTLR so it is first necessary to build Maven. Full instructions can be found at [Building-Maven](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven).

### Product Build - ANTLR 4.5.1
1. Set Environment Variables

    * Set JAVA_HOME and PATH

        * RHEL 7.1 and RHEL 6.6
		```
                export JAVA_HOME=/usr/lib/jvm/java
                export PATH=$PATH:$JAVA_HOME/bin
		```		

        * SLES 12 and SLES 11
		```
                export JAVA_HOME=/usr/lib64/jvm/java
                export PATH=$PATH:$JAVA_HOME/bin
		```


2. Download the ANTLR source code


		cd /<source_root>/
		git clone https://github.com/antlr/antlr4.git
        cd antlr4
		git checkout 4.5.1

		
3. Compile and install ANTLR using Maven

         mvn -Dgpg.skip=true -Duser.name="Your Username" -Dbootclasspath.compile=$JAVA_HOME/jre/lib/rt.jar -Djava6.home=$JAVA_HOME/jre install
		
4. Set CLASSPATH for ANTLR

        export CLASSPATH=$CLASSPATH:/<source_root>/antlr4/tool/target/antlr4-4.5.1.jar

5. Verify the build

    The following command should display the version of ANTLR that has been built and output usage information.

		java org.antlr.v4.Tool 

#### Known Issues:
* Based on speed of your processor build time may vary. By default *timeout* is set to "600000"ms in build.xml. Due to this you may face timeout issue. You can solve it by editing timeout to a higher value.

* If you get an OutofMemoryError during bootstrap, try setting an environment variable MAVEN_OPTS to provide more memory, e.g.

        MAVEN_OPTS=-XX:MaxPermSize=128m -Xmx512m

* BootClassPath may vary based on your rt.jar and other required Classes location, export the required classpath. You can export multiple classpaths with ':' separator

### References:
[ANTLR](http://www.antlr.org)
[Maven](https://git-wip-us.apache.org/repos/asf/maven.git)
[ANTLR4-download](https://github.com/antlr/antlr4)