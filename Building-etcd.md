<!---PACKAGE:etcd--->
<!---DISTRO:SLES 12:2.3.2--->
<!---DISTRO:RHEL 7.1:2.3.2--->

# Building etcd

etcd 2.3.2 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1 and SLES 12.

_**General Notes:**_ 	 
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Prerequisites (for RHEL 7.1 and SLES 12)
  * Go
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go).

### Building etcd
1. Install the build dependencies

    For RHEL 7.1
    ```shell
    sudo yum install curl wget tar 
    ```
    For SLES12
    ```shell
    sudo zypper install curl wget tar 
    
    ```
2. Get etcd v2.3.2 source code from github

    ```
    cd /<source_root>/
    $ wget https://github.com/coreos/etcd/archive/v2.3.2.tar.gz
    $ tar zxvf v2.3.2.tar.gz
    ```
3. Check go version

    ```
    $ go version
    ```
4. Build etcd

    ```
    cd /<source_root>/etcd-2.3.2
    $ ./build
    ```
5. Execute test suite
   ```
    cd /<source_root>/etcd-2.3.2
    ./test
   ```
_**Notes:**_  
_i) In case of error `“go test: -race and -msan are only supported on linux/amd64, freebsd/amd64,window"`, modify `/<source_root>/etcd-2.3.2/test` file and add support for "s390x" architecture as follows:_
    ```
    if [ $MACHINE_TYPE != "armv7l" ];[ $MACHINE_TYPE != "s390x" ];then
      RACE="--race"
    fi
    ```
_ii) If failure `cannot receive from ch as expected` is encountered, increase timeout in function 'TestWaitTime' in `./pkg/wait/wait_time_test.go` file as follows:_

    ```
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
   _**Note:** If the test case fails on timeout then increase the timeout limit._

   _iii) Few test case failures seem intermittent and should pass on multiple run._

6. Test etcd service

    ```
    cd /<source_root>/etcd-2.3.2
    ./bin/etcd
    ```

    _**Note:** In case of error `etcdmain: etcd on unsupported platform without ETCD_UNSUPPORTED_ARCH=s390x set"`, set following environment variable and rerun the command:_
    
    ```
    export ETCD_UNSUPPORTED_ARCH=s390x
    ```
    This will bring up etcd listening on port 2379 for client communication and on port 2380 for server-to-server communication.
 
  Next, let's set a single key, and then retrieve it:

    ```
    $ curl -L http://127.0.0.1:2379/v2/keys/mykey -XPUT -d value="this is awesome"
    $ curl -L http://127.0.0.1:2379/v2/keys/mykey
    ```

  You have successfully started an etcd and written a key to the store.
 
### References:
https://coreos.com/etcd/
