<!---PACKAGE:etcd--->
<!---DISTRO:SLES 12.x:3.0.15--->
<!---DISTRO:RHEL 7.x:3.0.15--->
<!---DISTRO:Ubuntu 16.x:3.0.15--->

# Building etcd

Below versions of etcd are available in the respective distributions at the time of this recipe creation: 

*    Ubuntu 16.04/16.10 has `2.2.5`

The instructions provided below specify the steps to build etcd v3.0.15 on Linux on the IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12/12-SP1 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	 
* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Prerequisites (for RHEL 7.1/7.2/7.3 and SLES 12/12-SP1)
  * Go
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7).

### Building etcd
1. Install the build dependencies

    For RHEL 7.1/7.2/7.3
    ```shell
    sudo yum install curl git wget tar gcc which
    ```

    For SLES 12/12-SP1
    ```shell
    sudo zypper install curl git wget tar gcc which
    
    ```
    For Ubuntu 16.04/16.10
    ```shell
    sudo apt-get update
    sudo apt-get install golang git curl
    ```
	
2. Get etcd v3.0.15 source code from github

    ```shell
    cd /<source_root>/
	mkdir -p /<source_root>/src/github.com/coreos
    cd /<source_root>/src/github.com/coreos
    git clone https://github.com/coreos/etcd
    cd etcd
    git checkout v3.0.15
    ```
3. Set GOPATH
   
    ```shell
	export GOPATH=/<source_root>/
	```
    
4. Check go version

    ```shell
    go version
    ```
5. Build etcd

    ```shell
    cd /<source_root>/src/github.com/coreos/etcd
    ./build
    ```
6. Execute test suite
   ```shell
    cd /<source_root>/src/github.com/coreos/etcd
    ./test
   ```
_**Notes:**_  
_* In case of error `"go test: -race and -msan are only supported on linux/amd64, freebsd/amd64,window"`, modify `/<source_root>/etcd/test` file and add support for "s390x" architecture as follows:_
    ```shell
    if [ $MACHINE_TYPE != "armv7l" ];[ $MACHINE_TYPE != "s390x" ];then
      RACE="--race"
    fi
    ```
_* If the test case fails on timeout then increase the timeout limit of the respective test case._
	
	_Example:_
	
	_If failure `cannot receive from ch as expected` is encountered, increase timeout in function 'TestWaitTime' in `./pkg/wait/wait_time_test.go` file as follows:_

    ```go
    func TestWaitTime(t *testing.T) {
     .
     .
     .
      wt.Trigger(t2)
      select {
      case <-ch2:
      case <-time.After(100 * time.Millisecond):
              t.Fatalf("cannot receive from ch as expected")
      }
     .
     .
     .
    ```

   _If failure `filenames = [.nfs00000000015847090007acea 7.test 8.test 9.test], want [7.test 8.test 9.test]` is encountered, increase timeout in function 'TestPurgeFileHoldingLock' in `./pkg/fileutil/purge_test.go` file as follows:_

   ```go
   func TestPurgeFileHoldingLock(t *testing.T) {
    .
    .
    .
     stop := make(chan struct{})
	 errch := PurgeFile(dir, "test", 3, time.Millisecond, stop)
	 time.Sleep(100 * time.Millisecond)
    .
    .
    .
   ```
	
   _**Note:** Few test case failures seem intermittent and should pass on multiple run._

7. Test etcd service

    ```shell
    cd /<source_root>/src/github.com/coreos/etcd
    ./bin/etcd
    ```

    _**Note:** In case of error `etcdmain: etcd on unsupported platform without ETCD_UNSUPPORTED_ARCH=s390x set"`, set following environment variable and rerun the command:_
    
    ```shell
    export ETCD_UNSUPPORTED_ARCH=s390x
    ```
    This will bring up etcd listening on port 2379 for client communication and on port 2380 for server-to-server communication.
 
  Next, let's set a single key, and then retrieve it:

    ```shell
    curl -L http://127.0.0.1:2379/v2/keys/mykey -XPUT -d value="this is awesome"
    curl -L http://127.0.0.1:2379/v2/keys/mykey
    ```

  You have successfully started an etcd and written a key to the store.
 
### References:
https://coreos.com/etcd/
