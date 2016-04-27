<!---PACKAGE:Scala--->
<!---DISTRO:SLES 12:2.11.8--->
<!---DISTRO:SLES 11:2.11.8--->
<!---DISTRO:RHEL 7.1:2.11.8--->
<!---DISTRO:RHEL 6.6:2.11.8--->

**Scala** can be installed on Linux on z Systems running RHEL 6.6/7.1 and SLES 11/12 by following these instructions. Version 2.11.8 of Scala has been successfully installed and tested this way.

_**General Notes:**_ 	

i) _When following the steps below please use a standard permission user unless otherwise specified._
	 
ii) _For convenience 'vi' has been used in the instructions below when editing files.  Replace with your desired editing program, if required._

iii) _A directory `/<source_root>/` will be referred to in these instructions.  This is a temporary writeable directory anywhere you'd like to place it._

## Installing Scala

###Obtain pre-built dependencies and create `/<source_root>/` directory.
    
   The main dependency for `Scala` is java  `1.6` or greater. Check on the command line for both  `java -version` and `javac -version`, and if either is not available, or reports a version less than  `1.6` an upgrade/install will be needed.

1. Install dependencies

	On RHEL 7.1 & 6.6	
	```shell    
 	sudo yum install wget java-1.7.1-ibm-devel
	```
	On SLES 11	
	```shell
	sudo zypper refresh
 	sudo zypper install wget java-1_7_0-ibm-devel
	```
	On SLES 12
	```shell    
	sudo zypper refresh	
 	sudo zypper install wget java-1_7_1-ibm-devel
	```
    
2. Create a working directory with write permission to use as a temporary installation workspace (Referred to as `/<source_root>/`)

	```shell
	mkdir /<source_root>/
	cd /<source_root>/
	```	

###Download and Install Scala

1. Download and install Scala using the following commands
	
	```shell
	wget http://downloads.typesafe.com/scala/2.11.8/scala-2.11.8.rpm
	sudo rpm -ivh scala-2.11.8.rpm
	```
2. *[Optionally]* Check Scala binary and compiler version
	
    Check scala version by using the following command
	```shell
	scala -version
	```
	The output of scala should be
	```shell
	Scala code runner version 2.11.8 -- Copyright 2002-2013, 	LAMP/EPFL
	```
	Check scala compiler version by using following command
	```shell
	scalac -version
	```
	The output of scala compiler version should be 
	```shell
	Scala compiler version 2.11.8 -- Copyright 2002-2013, 	LAMP/EPFL
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