## Building ANTLR 4.5.1

**ANTLR 4.5.1** has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7, RHEL 6, SLES 12 and SLES 11. More information on ANTLR is available at [ANTLR-website](http://www.antlr.org) and the source code can be downloaded from [ANTLR4-download](https://github.com/antlr/antlr4).

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
	   
### Dependency build - Ant 1.8
For RHEL 6.6 and SLES 11 build Ant 1.8 from source
		
		cd /<source_root>/
		git clone https://github.com/apache/ant.git
		cd ant
		git checkout ANT_182
		./build.sh
		
For RHEL 6.6 and SLES 11 set Environment Variables for Ant

		export ANT_HOME=/<source_root>/ant/dist
		export PATH=$PATH:$ANT_HOME/bin

### Dependency build - Maven 3.3.9
This recipe uses Maven (version >= 3.3.3) to build ANTLR so it is first necessary to build Maven. Full instructions can be found at [Building-Maven](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven).

### Product Build - ANTLR 4.5.1
1. Set Environment Variables

    * Set JAVA_HOME and PATH

        * RHEL 7.1
		```
                export JAVA_HOME=/usr/lib/jvm/java-1.7.1-ibm-1.7.1.3.10-1jpp.1.el7_1.s390x
                export PATH=$PATH:$JAVA_HOME/bin
		```		
        * RHEL 6.1
		```
                export JAVA_HOME=/usr/lib/jvm/java-1.7.1-ibm-1.7.1.3.10.s390x
                export PATH=$PATH:$JAVA_HOME/bin
		```		
        * SLES 12
		```
                export JAVA_HOME=/usr/lib64/jvm/java-1.7.1-ibm-1.7.1
                export PATH=$PATH:$JAVA_HOME/bin
		```
        * SLES 11
		```
                export JAVA_HOME=/usr/lib64/jvm/java-1.7.0-ibm-1.7.0
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
You can verify installation and check version by running

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


> 	The information provided in this article is accurate at the time of writing, but on-going development in the 
> 	open-source projects involved may make the information incorrect or obsolete. Please contact us on [developerWorks](https://www.ibm.com/developerworks/community/groups/community/lozopensource)
> 	if you have any questions or feedback.