<!---PACKAGE:cAdvisor--->
<!---DISTRO:SLES 12:0.23.2--->
<!---DISTRO:RHEL 7.1:0.23.2--->
<!---DISTRO:Ubuntu 16.x:0.23.2--->

# Building cAdvisor

Below versions of cAdvisor are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `0.20.5`

The instructions provided below specify the steps to build cAdvisor version 0.23.2 on Linux on the IBM z Systems for RHEL 7.1, SLES 12 and Ubuntu 16.04.

_**General Notes:**_   
_i) When following the steps below please use a standard permission user unless otherwise specified._     
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Prerequisites:
* go (Refer [go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go) recipe)  _For Ubuntu, use apt-get to install golang package from the repository._
* git


### Building and Installing cAdvisor

1. Install following dependency

   RHEL 7.1: 
   ```
         sudo yum install -y git
    ```
  
   SLES 12:
    ```
        sudo zypper install -y git 
    ```
	
	Ubuntu 16.04:
    ```
		sudo apt-get update
		sudo apt-get install -y git golang 
    ```
	
2. Export go path
 
   ```
        export GOPATH=/<source_root>/ 
        export PATH=$PATH:$GOPATH/bin:$HOME/go/bin

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
         git clone https://github.com/google/cadvisor.git -b v0.23.2
    ```

7. Change the work directory
    ```  
         cd $GOPATH/src/github.com/google/cadvisor
    ```
8. Modify `$GOPATH/src/github.com/google/cadvisor/Godeps/_workspace/src/github.com/klauspost/crc32/crc32.go` to add the following code to end of the file
 
	```
		func updateCastagnoli(crc uint32, p []byte) uint32 {
				// only use slicing-by-8 when input is >= 16 Bytes
				if len(p) >= 16 {
						return updateSlicingBy8(crc, castagnoliTable8, p)
				}
				return update(crc, castagnoliTable, p)
		}

		func updateIEEE(crc uint32, p []byte) uint32 {
				// only use slicing-by-8 when input is >= 16 Bytes
				if len(p) >= 16 {
						iEEETable8Once.Do(func() {
								iEEETable8 = makeTable8(IEEE)
						})
						return updateSlicingBy8(crc, iEEETable8, p)
				}
				return update(crc, IEEETable, p)
		}
	```
	
	_**Note:**_ 
    cAdvisor uses 'klauspost/crc32' over standard crc32 library of golang which is optimized for x64 platform. We add the above code to align it with Go standard library hash/crc32.
	
9. Build cAdvisor
    ```
         godep go build .
    ```

10. (Optional) Run unit tests
    ```
         cd $GOPATH/src/github.com/google/cadvisor 
         godep go test ./... -test.short
    
    ```

   _**Note:**_ 
    Ignore `TestTopology` failure as it is known issue on system Z.


11. Run cAdvisor
    ```
        sudo ./cadvisor
    ```
	
12. Access cAdvisor web user interface from browser
    ```
        http://<host-ip>:<http-port>/
    ```

  _**Note:**_ 
    Default port number for cAdvisor is `8080`

	
#### References:
* https://github.com/google/cadvisor