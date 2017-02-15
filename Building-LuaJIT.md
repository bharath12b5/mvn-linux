<!---PACKAGE:Scala--->
<!---DISTRO:SLES 12.x:2.1--->
<!---DISTRO:SLES 11.x:2.1--->
<!---DISTRO:RHEL 7.x:2.1--->
<!---DISTRO:Ubuntu 16.x:2.1--->

# Building LuaJIT

The instructions provided below specify the steps to build LuaJIT version 2.1 on IBM z Systems for following distributions:

* SLES (11 SP4, 12, 12 SP1, 12 SP2)
* RHEL (7.1, 7.2, 7.3)
* Ubuntu (16.04, 16.10)

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##Step 1: Building and Installing LuaJIT

####1.1) Install dependencies

* SLES (11 SP4, 12, 12 SP1, 12 SP2)
    ```bash
	sudo zypper update
    sudo zypper install git gcc make
    ```
	
* RHEL (6.8,7.1,7.2,7.3)	
    ```bash
    sudo yum install gcc make git
	```
	
* Ubuntu (16.04 , 16.10)	
    ```bash
    sudo apt-get update
    sudo apt-get install git gcc make
	```

####1.2) Download  LuaJIT 
  ```bash
  cd /<source_root>/
  git clone https://github.com/linux-on-ibm-z/LuaJIT.git
  cd LuaJIT
  git checkout v2.1
  ```

####1.3) Build and Install LuaJIT
  ```bash
  cd /<source_root>/LuaJIT/
  make
  sudo make install
  sudo ln -s luajit-2.1.0-beta2 /usr/local/bin/luajit
  ```

##References:
http://luajit.org/ 
