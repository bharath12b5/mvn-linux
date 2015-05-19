#Building AntLR 3.5

The following build instructions have been tested with **AntLR 3.5** on SLES12 and RHEL7 on IBM z Systems.

###Step 1:Install the Dependencies
Following are the build dependencies for installing AntLR. Maven is not available in SLES12, RHEL7 repository, so we shall build it from source

*       java-1.7.0-openjdk-devel(RHEL7) / java-1_7_0-openjdk-devel(SLES12)
*		git (RHEL7) / git-core (SLES12)
*       maven

**Inter Dependency to build maven**

ANT is additional dependency to build maven
* ant

To install packages use yum install on RHEL7 and zypper install on SLES12. For example:
 
RHEL7:
```	
	yum install -y git java-1.7.0-openjdk-devel.s390x ant
```
SLES12:
```	
	zypper install -y git-core java-1_7_0-openjdk-devel ant
```		   
### Step 2: Set Environment variables

1. Set JAVA_HOME and PATH

    RHEL7:

        export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk
        export PATH=$PATH:$JAVA_HOME/bin

    SLES12:

        export JAVA_HOME=/usr/lib64/jvm/java-1.7.0-openjdk-1.7.0
        export PATH=$PATH:$JAVA_HOME/bin

2. Set Maven Home and PATH

        export M2_HOME=/maven_build
        export PATH=$PATH:/$M2_HOME/bin

3. Set CLASSPATH for AntLR

        export $WORK_DIR=`pwd`
        export CLASSPATH=$CLASSPATH:$WORK_DIR/antlr3/antlr-complete/target/antlr-complete-3.5.jar


### Step 3: Building Maven(Dependency) :
Download  source from git and build maven: [Maven]

		git clone https://git-wip-us.apache.org/repos/asf/maven.git/ --branch maven-3.2.5
        cd maven
        ant

Once build is complete you will could see a build successful message.

###Step 4:Building AntLR3.5:

Download  source from git: [AntLR]

		git clone --branch antlr-3.5 https://github.com/antlr/antlr3.git
        cd antlr3

**Compile and install antlr using maven**

    mvn -Dgpg.skip=true -Duser.name="Your Username" -Dbootclasspath.compile=$JAVA_HOME/jre/lib/rt.jar -Djava6.home=$JAVA_HOME/jre install

###Verification:
You can verify installation by running

    java org.antlr.Tool --help

####Known Issues:
* Based on speed of your processor build time may vary. By default *timeout* is set to "600000"ms in build.xml. Due to this you may face timeout issue. You can solve it by editing timeout to a higher value.

* If you get an OutofMemoryError during bootstrap, try setting an environment variable MAVEN_OPTS to provide more memory, e.g.

        MAVEN_OPTS=-XX:MaxPermSize=128m -Xmx512m

* BootClassPath may vary based on your rt.jar and other required Classes location, export the required classpath. You can export multiple classpaths with ':' separator

##References:
    https://github.com/antlr/antlr3/releases

[Maven]:https://git-wip-us.apache.org/repos/asf/maven.git/
[AntLR]:https://github.com/antlr/antlr3.git
