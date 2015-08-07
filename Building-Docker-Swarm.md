# Building Docker Swarm
The following build instructions describe how to build Docker Swarm on RHEL7 and SLES12 for IBM z Systems.

### Version
0.3.0

### Section 1: Install the following Dependencies
* gccgo (Refer gccgo recipe file)
* git

RHEL7:
```
    yum install -y  git
```
SLES12:
```
    zypper install -y git
```

### Section 2.  Set GOPATH environment variable

         export GOPATH=/go/src/github.com/docker/swarm/Godeps/_workspace:/go
         export PATH=$PATH:$GOPATH/bin
    
### Section 3. Download the source and install swarm

3.1 .  Get dependent tools
``` 
    go get github.com/tools/godep
```
3.2 Download the source from gihub
```
    git clone -b v0.3.0 https://github.com/docker/swarm.git
```
3.3 Move/Copy to required location
```
    cp -r -n swarm/* /go/src/github.com/docker/swarm
```
3.4 Install Docker Swarm
```
    go install github.com/docker/swarm/
```

## References:

https://github.com/docker/swarm.git

                                                                                                                                         