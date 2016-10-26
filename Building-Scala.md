<!---PACKAGE:Scala--->
<!---DISTRO:SLES 12.x:2.11--->
<!---DISTRO:SLES 11.x:2.11--->
<!---DISTRO:RHEL 7.x:2.11--->
<!---DISTRO:RHEL 6.x:2.11--->
<!---DISTRO:Ubuntu 16.x:Distro, 2.11--->

# Building Scala

Below versions of Scala are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `2.11.6`

The instructions provided below specify the steps to build Scala version 2.11.8 on Linux on the IBM z Systems for RHEL 6.7/7.1/7.2, SLES 11-SP3/12/12-SP1 and Ubuntu 16.04. 

_**General Notes:**_ 	

i) _When following the steps below please use a standard permission user unless otherwise specified._
	 
ii) _For convenience 'vi' has been used in the instructions below when editing files.  Replace with your desired editing program, if required._

iii) _A directory `/<source_root>/` will be referred to in these instructions.  This is a temporary writeable directory anywhere you'd like to place it._

## Installing Scala

###Obtain pre-built dependencies and create `/<source_root>/` directory.
    
   The main dependency for `Scala` is java  `1.6` or greater. Check on the command line for both  `java -version` and `javac -version`, and if either is not available, or reports a version less than  `1.6` an upgrade/install will be needed.

1. Install dependencies

	On RHEL 7.1/7.2 & 6.7	
	```shell    
 	sudo yum install wget java-1.7.1-ibm-devel
	```
	On SLES 11-SP3	
	```shell
	sudo zypper refresh
 	sudo zypper install wget java-1_7_0-ibm-devel
	```
	On SLES 12/12-SP1
	```shell    
	sudo zypper refresh	
 	sudo zypper install wget java-1_7_1-ibm-devel
	```
	On Ubuntu 16.04
	```shell    
	sudo apt-get update	
 	sudo apt-get install wget openjdk-8-jdk
	```
    
2. Create a working directory with write permission to use as a temporary installation workspace (Referred to as `/<source_root>/`)

	```shell
	mkdir /<source_root>/
	cd /<source_root>/
	```	

###Download and Install Scala

1. Download and install Scala using the following commands
	
	On Rhel 6.7/7.1/7.2 & SLES 11-SP3/12/12-SP1
	```shell
	wget http://www.scala-lang.org/files/archive/scala-2.11.8.rpm
	sudo rpm -ivh scala-2.11.8.rpm
	```
	On Ubuntu 16.04
	```shell
	wget http://www.scala-lang.org/files/archive/scala-2.11.8.deb
	sudo dpkg -i scala-2.11.8.deb
	```
2. *[Optionally]* Check Scala binary and compiler version
	
    Check scala version by using the following command
	```shell
	scala -version
	```
	The output of scala should be
	```shell
	Scala code runner version 2.11.8 -- Copyright 2002-2016, LAMP/EPFL
	```
	Check scala compiler version by using following command
	```shell
	scalac -version
	```
	The output of scala compiler version should be 
	```shell
	Scala compiler version 2.11.8 -- Copyright 2002-2016, LAMP/EPFL
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