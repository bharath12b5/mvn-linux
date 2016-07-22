<!---PACKAGE:Alfresco--->
<!---DISTRO:SLES 12:5.1--->
<!---DISTRO:SLES 11:5.1--->
<!---DISTRO:RHEL 7.1:5.1--->
<!---DISTRO:RHEL 6.6:5.1--->
<!---DISTRO:Ubuntu 16.x:5.1--->

## Building Alfresco

The instructions provided below specify the steps to build [Alfresco](https://www.alfresco.com/) version 5.1 on Linux on the IBM z Systems for RHEL 7.1, SLES 12(sp1), RHEL 6, SLES 11 and Ubuntu 16.04


_**General Notes:**_ 	 
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

1. Install dependencies

    * RHEL 7.1
		
		With IBM JDK:
		
			
			sudo yum install -y wget tar subversion java-1.8.0-ibm-devel.s390x java-1.8.0-ibm.s390x
			
		With Open JDK:
		
			
			sudo yum install -y wget tar subversion java-1.8.0-openjdk.s390x java-1.8.0-openjdk-devel.s390x java-1.8.0-openjdk-headless.s390x
			
		
    * SLES 12(sp1)
		
		With IBM JDK:
		
			
			sudo zypper install -y wget tar subversion java-1_8_0-ibm java-1_8_0-ibm-devel 
			
		
		With Open JDK:
		
			sudo zypper install -y wget tar subversion java-1_8_0-openjdk java-1_8_0-openjdk-devel java-1_8_0-openjdk-headless
		
	* RHEL 6
	
		```
		sudo yum install -y wget tar subversion  java-1.8.0-ibm.s390x  java-1.8.0-ibm-devel.s390x
		```
		
	* SLES 11
	
		```
		sudo zypper install -y wget tar subversion
		```
	
		* Install IBM Java 8
		
			Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.
		

		 
		
	* Ubuntu 16.04
	
		With IBM JDK:
			
			sudo apt-get install -y wget tar subversion
			
			* Install IBM Java 8
		
			  Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link.
		


			
		
		With Open JDK:
		
			sudo apt-get install -y wget tar subversion openjdk-8-jdk
			
2.  Install Maven

		wget https://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
		tar -xvzf apache-maven-3.3.9-bin.tar.gz

3. Get Alfresco Source code
    
        cd  /<source_root>/
        svn co https://svn.alfresco.com/repos/alfresco-open-mirror/alfresco/COMMUNITYTAGS/5.1.g/
		
4. Set Environment Variables

		export M2_HOME=<path to apache-maven>
		export PATH=$M2_HOME/bin:$JAVA_HOME/bin:$PATH
		export JAVA_HOME=<path to java>
		export MAVEN_OPTS="-Xmx512m -XX:MaxPermSize=1G"

5. Build and Install Alfresco (Skipping Test cases)
		
		cd /<source_root>/5.1.g/root/
		mvn clean install -DskipTests -fn		
        
   On successful build following wars will be created on the below mentioned paths

    * `/<source_root>/5.1.g/root/projects/web-client/target/alfresco.war`
    * `/<source_root>/5.1.g/root/projects/server-root/target/ROOT.war`
    * `/<source_root>/5.1.g/root/projects/solr4/target/solr4.war`
    * `/<source_root>/5.1.g/root/projects/solr/source/solr/instance/apache-solr-1.4.1.war`
    * `/<source_root>/5.1.g/root/projects/solr/target/solr.war`
		
	Note:
	Incase you experience stackoverflow issues while executing test cases in data-model module,
	add the following plugin in pom.xml file 
	`<source_root>/5.1.g/root/projects/data-model/pom.xml`
	
	
		<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.2</version>
				<configuration>
					<compilerId>eclipse</compilerId>
					<optimize>true</optimize>
				</configuration>
				<dependencies>
					<dependency>
						<groupId>com.xqbase</groupId>
						<artifactId>xqbase-compiler-eclipse</artifactId>
						<version>0.1.0</version>
					</dependency>
				</dependencies>
		</plugin>

6.  Testing ( Optional )

		cd /<source_root>/5.1.g/root/
		mvn clean install -fn
		
	It was observed that some test cases failed when executed from top of the source tree.
	All these test cases passed when executed individually.
	
	Note:
	If you are using IBM JDK following test case failures will be seen. These are not related to IBM z Systems.
	
	* EncryptorTest
	* KeyStoreKeyProviderTest
	* MLAnalayserTest
	
### Reference:

https://www.alfresco.com/	