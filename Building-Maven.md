#Building Maven 3.2.5

The following build instructions have been tested with **Maven 3.2.5** on SLES12 and RHEL7 on IBM z Systems.

####Step 1:Install the Dependencies
Following are the build dependencies for installing Maven.
*       java-1.7.0-openjdk-devel(on RHEL7) / java-1_7_0-openjdk-devel(on SLES12)
*		git (RHEL7) / git-core (SLES12)
*       ant

RHEL7:
```
yum -y update && yum install -y \
   git\
   java-1.7.0-openjdk-devel.s390x \
   ant
```

SLES12:
```
zypper install -y \
   git-core \
   java-1_7_0-openjdk-devel \
   ant
```
####Step 2: Clone the repository and checkout version 3.2.5
Download  source from git: [Maven]
	git clone https://git-wip-us.apache.org/repos/asf/maven.git/ --branch maven-3.2.5

####Step 3:Setting Environment variables
1.Set JAVA_HOME and PATH

On RHEL7:

    export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk
    export PATH=$PATH:$JAVA_HOME/bin

On SLES12:

    export JAVA_HOME=/usr/lib64/jvm/java-1.7.0-openjdk-1.7.0
    export PATH=$PATH:$JAVA_HOME/bin

2.Set Maven Home and PATH

    export M2_HOME=/maven_build
    export PATH=$PATH:/$M2_HOME/bin

####Step 4: Build Maven
**ANT** is used to build maven. Test Cases are included as part of build.

    cd maven
    ant

Once build is complete you will see a build successful message.

###Verification:
Run command `mvn --help` to verify installation.

####Known Issues:
* Based on speed of your processor build time may vary. By default *timeout* is set to "600000"ms in build.xml. Due to this you may face timeout issue. You can solve it by editing timeout to required value.

* If you get an OutofMemoryError during bootstrap, try setting an environment variable MAVEN_OPTS to provide more memory, e.g.

        MAVEN_OPTS=-XX:MaxPermSize=128m -Xmx512m

##References:
http://maven.apache.org/archives/maven-1.x/index.html

[Maven]:https://git-wip-us.apache.org/repos/asf/maven.git/
