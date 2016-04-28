<!---PACKAGE:Docker Swarm--->
<!---DISTRO:SLES 12:1.1.0--->
<!---DISTRO:RHEL 7.1:1.1.0--->
<!---DISTRO:Ubuntu 16.x:1.1.0--->

# Building Docker Swarm
Docker Swarm 1.1.0 can be built and tested on Linux on z Systems (RHEL 7.1, SLES 12 and Ubuntu 16.04) by following these instructions.

_**General Notes:**_  
_i) When following the steps below please use a standard permission user unless otherwise specified._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._   

### Section 1: Install the following Dependencies
* go (Refer [go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go) recipe)  _For Ubuntu, use apt-get to install golang package from the repository._
* git

RHEL7:
```
    sudo yum install git
```

SLES12:
```
    sudo zypper install git
```

Ubuntu 16.04:
```
	sudo apt-get install git golang
```

_**Note:**_ 
_Git version 1.8.5 or greater is compatible with Go version 1.5.2. Check installed git version and if the installed git version is lesser than 1.8.5 then build it from source using the steps given below._

  1. Check the version of existing git executable.
    ```
  git --version
```   
_**Note:** If above command gives git version lesser than 1.8.5, then follow below steps, else jump to Section 2 i.e. **'Set GOPATH environment variable'**._  

  2. Install build dependencies for git.
    ```
  sudo yum install -y asciidoc-8.6.8-5.el7.noarch openssl-devel curl-devel expat-devel perl-ExtUtils-MakeMaker gettext-devel tar make
```  

  3. Download the source from github.
    ```
  cd /<source_root>/
  git clone https://github.com/git/git.git
  cd /<source_root>/git/
  git checkout v1.8.5
```

  4. Build and Install git.
    ```
  make prefix=/usr/local && sudo make prefix=/usr/local install 
```

  5. Export git binary path to PATH environment variable.
    ```
  export PATH=/usr/local/bin:$PATH  
```

### Section 2:  Set GOPATH environment variable

       export GOPATH=/<source_root>/
       export PATH=$HOME/go/bin:$PATH:$GOPATH/bin
       export GOPATH=$GOPATH:$GOPATH/src/github.com/docker/swarm/Godeps/_workspace/
    
### Section 3: Download the source and install Docker Swarm

1.  Get dependent tools

   ``` 
    cd /<source_root>/
    go get github.com/tools/godep
   ```
2.  Download the source from github

   ```
    mkdir -p /<source_root>/src/github.com/docker
    cd /<source_root>/src/github.com/docker
    git clone https://github.com/docker/swarm.git
    cd swarm 
    git checkout v1.1.0
   ```
3.  Install Docker Swarm

   ```
    go install github.com/docker/swarm/
   ```

### Section 4: Starting the swarm manager and registering nodes
 
 1. A node should be listening to the tcp port of swarm manager. First start docker with -H flag on the node
      ```
    docker -H tcp://0.0.0.0:2375 -d  
      ```
	
 2. Create a static file 'mycluster' in `<source_root>` on the swarm manager, containing the IP address and docker port of the nodes. Keep the file contents in the following format:

	
           node-ip1:port
           node-ip2:port
           node-ip3:port
        
   
 3. Start the swarm manager
       ```
    swarm manage --host=0.0.0.0:2375  file://<source_root>/mycluster
       ```	
    You should see log messages like:
      ```
    Listening for HTTP                        addr=0.0.0.0:2375 proto=tcp

    Registered engine	                      at <node-ip>
      ```
	on the stdout
	
 4. Once the Setup is complete You can check if you are able to manage the nodes using regular docker commands
	```
	docker -H tcp://<swarm_ip:swarm_port> ps
	docker -H tcp://<swarm_ip:swarm_port> info
	```
	where  *swarm_ip*: is the ip on which manager has been started
		   *swarm_port*: the port used while starting the manager


## References:

https://github.com/docker/swarm.git

                                                                                                                                        