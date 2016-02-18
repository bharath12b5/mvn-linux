# Building etcd

etcd v2.2.5 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1 and SLES 12.

### Prerequisites
  * Go
  -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go).

### Building etcd
1. Get etcd v2.2.5 source code from github
    ```
    $ wget https://github.com/coreos/etcd/archive/v2.2.5.tar.gz
    ```

1. Untar and cd to the work directory
    ```
    $ tar zxvf v2.2.5.tar.gz
    $ cd etcd-2.2.5
    ```

1. Check go version
    ```
    $ go version
    ```

1. Adjust build script if needed

  If your go version is something like "devel +xxxxxxx" rather a release number. You'll need to adjust the build script.

  Edit file "build".

    ```
    $ vi build
    ```

   Add "ver=6" after line "ver=$(echo $val | awk -F ' ' '{print $3}' | awk -F '.' '{print $2}')".
    ```
      ver=$(echo $val | awk -F ' ' '{print $3}' | awk -F '.' '{print $2}')
      ver=6
    ```

1. Build etcd
    ```
    $ ./build
    ```

1. Test etcd

  First start a single-member cluster of etcd:

    ```
    $ ./bin/etcd
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