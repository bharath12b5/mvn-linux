# Building Heapster

The instructions provided below specify the steps to build Heapster 1.1.0 on IBM z Systems for RHEL 7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10

### Prerequisites
  * Go (RHEL 7.2/7.3 & SLES 12-SP1/12-SP2)
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7)

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified_

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_

#### Step 1: Install the Dependencies

* RHEL 7.2/7.3

        sudo yum install -y git make wget gcc tar

* SLES 12-SP1/12-SP2

        sudo zypper install -y git make wget gcc tar
		
* Ubuntu 16.04/16.10
		
		sudo apt-get install -y golang git make tar
		
#### Step 2: Build and install Heapster

 * Setup Environment variable and source folder to build the package

        export GOPATH=/<source_root>/
        export PATH=$GOPATH/bin:$PATH
        mkdir -p /<source_root>/src/k8s.io/     

 * Download Heapster source code

        cd /<source_root>/src/k8s.io/
        git clone https://github.com/linux-on-ibm-z/heapster.git
        cd heapster
        git checkout v1.1.0-s390x

* Checkout the dependency `boltdb/bolt v1.3.0` otherwise build will fail with `undefined: maxMapSize & invalid array bound maxMapSize`

        rm -rf /<source_root>/src/k8s.io/heapster/vendor/github.com/boltdb/bolt
        cd /<source_root>/src/k8s.io/heapster/vendor/github.com/boltdb/
        git clone https://github.com/boltdb/bolt.git
        cd bolt
        git checkout v1.3.0

* Build Heapster

        cd /<source_root>/src/k8s.io/heapster
        make ARCH=s390x

* Run test cases(Optional)
        
        cd /<source_root>/src/k8s.io/heapster
        make test-unit test-unit-cov ARCH=s390x

_**Note:** Test case "TestMonascaUnhealthy", failure is observed, change the URL as below for test case to pass. This change is going to be available in next release._
      
  Edit file `/<source_root>/src/k8s.io/heapster/metrics/sinks/monasca/monasca_client_test.go`

```diff
@@ -104,7 +104,7 @@
func TestMonascaUnhealthy(t *testing.T) {
    // setup
    ksClientMock := new(keystoneClientMock)
-   monURL, _ := url.Parse("http://unexisting.monasca.com")
+   monURL, _ := url.Parse("http://127.0.0.1:9")
    sut := &ClientImpl{ksClient: ksClientMock, monascaURL: monURL}
    ksClientMock.On("GetToken").Return(testToken, nil).Once()
```
