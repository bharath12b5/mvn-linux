<!---PACKAGE:Docker Swarm--->
<!---DISTRO:SLES 12:1.1.0--->
<!---DISTRO:RHEL 7.1:1.1.0--->

# Building Docker Swarm
Docker Swarm 1.1.0 can be built and tested on Linux on z Systems (RHEL 7.1 ans SLES 12) by following these instructions.

_**General Notes:**_  
_i) When following the steps below please use a standard permission user unless otherwise specified._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._   

### Section 1: Install the following Dependencies
* go (Refer to [go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go) recipe)
* git

RHEL7:
```
    sudo yum install -y git
```

SLES12:
```
    sudo zypper install -y git
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

3.1 .  Get dependent tools.
``` 
    cd /<source_root>/
    go get github.com/tools/godep
```
3.2 Download the source from github.
```
    mkdir -p /<source_root>/src/github.com/docker
    cd /<source_root>/src/github.com/docker
    git clone -b v1.1.0 https://github.com/docker/swarm.git
```
3.4 Install Docker Swarm.
```
    go install github.com/docker/swarm/
```

## References:

https://github.com/docker/swarm.git

                                                                                                                                        