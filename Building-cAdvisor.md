<!---PACKAGE:cAdvisor--->
<!---DISTRO:SLES 12:0.20.5--->
<!---DISTRO:RHEL 7.1:0.20.5--->

# Building cAdvisor

cAdvisor (version 0.20.5) has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1 and SLES 12.


### Prerequisites:
  * go
  -- You can get all the instructions for building go [from here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go)

### _**General Note:**_
_i) When following the steps below please use a standard permission user unless otherwise specified._     
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Building and Installing cAdvisor
1. Export go path
 
   ```
        export GOPATH=/<source_root>/ 
        export PATH=$PATH:$GOPATH/bin:$HOME/go/bin

   ```
 

2. Install following dependency

   RHEL7: 
   ```
         sudo yum install -y git
    ```
  
   SLES12:
    ```
        sudo zypper install -y git 
    ```
3. Install godep tool 
    ```
         cd /<source_root>/
         go get github.com/tools/godep
    ```

4. Create directory 
    ```
         mkdir -p $GOPATH/src/github.com/google
    ```

5. Change the work directory
    ```
         cd $GOPATH/src/github.com/google
    ```

6. Checkout the code from repository
    ```
         git clone https://github.com/google/cadvisor.git -b v0.20.5
    ```

7. Change the work directory
    ```  
         cd $GOPATH/src/github.com/google/cadvisor
    ```

8. Build cadvisor
    ```
         godep go build .
    ```

9. (Optional) Run unit tests
    ```
         cd $GOPATH/src/github.com/google/cadvisor 
         godep go test ./... -test.short
    
    ```

   _**Note:**_ 
    Ignore `TestTopology` failure as it is known issue on system Z.


10. Run cAdvisor
    ```
        sudo ./cadvisor
    ```
	
11. Access cAdvisor web user interface from browser
    ```
        http://<host-ip>:<http-port>/
    ```

  _**Note:**_ 
    Default port number for cAdvisor is `8080`
