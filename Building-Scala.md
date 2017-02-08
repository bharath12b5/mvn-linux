<!---PACKAGE:Scala--->
<!---DISTRO:SLES 12.x:2.12--->
<!---DISTRO:SLES 11.x:2.12--->
<!---DISTRO:RHEL 7.x:2.12--->
<!---DISTRO:RHEL 6.x:2.12--->
<!---DISTRO:Ubuntu 16.x:Distro, 2.12--->

# Building Scala

Below versions of Scala are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `2.11.6`
*    Ubuntu 16.10 has `2.11.8`

The instructions provided below specify the steps to build Scala version 2.12.1 on the IBM z Systems for
* RHEL (6.8,7.1,7.2,7.3)
* SLES (11 SP4, 12, 12 SP1, 12 SP2) 
* Ubuntu (16.04, 16.10) 

_**General Notes:**_ 	

i) _When following the steps below please use a standard permission user unless otherwise specified._
	 
ii) _For convenience 'vi' has been used in the instructions below when editing files.  Replace with your desired editing program, if required._

iii) _A directory `/<source_root>/` will be referred to in these instructions.  This is a temporary writeable directory anywhere you'd like to place it._

## Installing Scala

###Obtain pre-built dependencies and create `/<source_root>/` directory.
    
   The main dependency for `Scala` is java  `1.8` or greater. Check on the command line for both  `java -version` and `javac -version`, and if either is not available, or reports a version less than  `1.8` an upgrade/install will be needed.

1. Install dependencies

	On RHEL 6.8
      
       * With IBM JDK
 
       Install IBM Java 8
	 Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link
	
		```shell    
		sudo yum install wget 
		```


	On RHEL (7.1, 7.2, 7.3)
      
       * With IBM JDK
	
		```shell    
		sudo yum install wget java-1.8.0-ibm-devel
		```
		
	* With Open JDK
	
		```shell    
		sudo yum install wget java-1_8_0-openjdk-devel
		
		```

	On SLES (11 SP4, 12)
      
       * With IBM JDK
 
       Install IBM Java 8
	 Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link
	
		```shell    
		sudo zypper install wget 
		```


	On SLES (12 SP1, 12 SP2)
       * With IBM JDK
	
		```shell    
		sudo zypper install wget  java-1_8_0-ibm-devel

		```
		
	* With Open JDK
	
		```shell    
		sudo zypper install -y wget java-1_8_0-openjdk-devel
		
		```

	On Ubuntu (16.04, 16.10)

	* With Open JDK

	     ```shell    
	        sudo apt-get update	
 	        sudo apt-get install wget openjdk-8-jdk
	     ```

    * With IBM JDK    
	 
	 Install IBM Java 8
	 Download IBM Java 8 sdk binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link
	 
    
2. Create a working directory with write permission to use as a temporary installation workspace (Referred to as `/<source_root>/`)

	```shell
	mkdir /<source_root>/
	cd /<source_root>/
	```	

###Download and Install Scala

1. Download and install Scala using the following commands
	
	On RHEL ( 6.8 7.1, 7.2, 7.3) & SLES (11 SP4, 12, 12 SP1 12 SP2)
	```shell
	wget http://www.scala-lang.org/files/archive/scala-2.12.1.rpm
	sudo rpm -ivh scala-2.12.1.rpm
	```
	On Ubuntu (16.04, 16.10)
	```shell
	wget http://www.scala-lang.org/files/archive/scala-2.12.1.deb
	sudo dpkg -i scala-2.12.1.deb
	```
2. *[Optionally]* Check Scala binary and compiler version
	
    Check scala version by using the following command
	```shell
	scala -version
	```
	The output of scala should be
	```shell
	Scala code runner version 2.12.1 -- Copyright 2002-2016, LAMP/EPFL
	```
	Check scala compiler version by using following command
	```shell
	scalac -version
	```
	The output of scala compiler version should be 
	```shell
	Scala compiler version 2.12.1 -- Copyright 2002-2016, LAMP/EPFL
	```

## [Optional] Testing Scala Sample Program
    
1. Create a Test Program

    ```shell
    vi HelloWorld.scala
    ```
    Add the following contents
    ```scala
    object HelloWorld
    {
        def main(args: Array[String]) {
            println("Hello, world!")
        }
    }
    ```
    _**Note:** this simple test has been condensed from the 
[Getting Started With Scala](http://www.scala-lang.org/documentation/getting-started.html) examples_

2. Compile the Test Program

    ```shell
    scalac HelloWorld.scala
    ```
    _**Note:** this creates two compiled files: `HelloWorld$.class` and `HelloWorld.class`_
    
3. Execute the test program

    ```shell
    scala HelloWorld
    ```
    The output of the above test program should be
    ```shell
    Hello, world!
    ```

##References    
http://www.scala-lang.org/
