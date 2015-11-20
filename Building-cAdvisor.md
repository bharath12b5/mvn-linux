# Building cAdvisor

cAdvisor has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1 and SLES 12.

Note: cAdvisor pull request [#886](https://github.com/google/cadvisor/pull/886) has been created on GitHub. The code changes will be available in the next release of cAdvisor.


### Prerequisites:
  * gccgo
  -- You can get all the instructions for building gccgo [from here](https://github.com/linux-on-ibm-z/docs/wiki/Building-gccgo?cm_mc_uid=60971096199214062909410&cm_mc_sid_50200000=1438603503)

### _**General Note:**_
* _When following the steps below please use a standard permission user unless otherwise specified._

### Building and Installing cAdvisor
1. Export go path
    ```
        export GOPATH=/gcc/gcc/go; export PATH=$PATH:$GOPATH/bin
    ```
2. Install following dependencies
 
    *  git
    
   ```
        RHEL7:
            $ yum install --nogpgcheck -y git
        SLES12:
            $ zypper install -y git 
    ```

3. Install godep tool 
    ```
        $ go get github.com/tools/godep
    ```

4. Create directory 
    ```
        $ mkdir -p $GOPATH/src/github.com/google
    ```

5. Change the work directory
    ```
        $ cd /gcc/gcc/go/src/github.com/google
    ```

6. Checkout the code from repository
    ```
        $ git clone https://github.com/linux-on-ibm-z/cadvisor.git
    ```

7. Change the work directory
    ```  
        $ cd $GOPATH/src/github.com/google/cadvisor
    ```

8. Build cadvisor
    ```
        $ godep go build .
    ```

9. Run cAdvisor
    ```
        $ ./cadvisor
    ```
