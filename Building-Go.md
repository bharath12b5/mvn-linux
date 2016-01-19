These instructions describe how to build the latest development version of the [Go programming language](http://www.golang.org) for Linux on IBM z Systems running SLES 12 or RHEL 7.x.

_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified._

Building Go for Linux on z Systems is a two-stage process:

1. Cross-compile the Go bootstrap tool on an Intel/AMD-based machine running an up-to-date version of Linux
2. Build the Go tool-chain on Linux on z Systems with the bootstrap tool

For more generic information on how the Go bootstrapping process works take a look at this [blog entry](http://dave.cheney.net/2015/10/16/bootstrapping-go-1-5-on-non-intel-platforms).

## Cross-compiling the Go bootstrap tool

To build the Go bootstrap tool you will need to use an Intel/AMD-based machine running an up-to-date version of Linux. This guide has been tested on SLES 12, RHEL 7.0 and RHEL 7.1.

1. Create a directory for the amd64 version of the Go tool-chain:

    ```
mkdir $HOME/go1.5.2
cd $HOME/go1.5.2
    ```

1. Download the amd64 Go tool-chain binary and extract it:

    ```
wget https://storage.googleapis.com/golang/go1.5.2.linux-amd64.tar.gz
tar xvfz go1.5.2.linux-amd64.tar.gz
    ```

1. Clone the source code of the z Systems port of Go:

    ```
cd $HOME
git clone http://github.com/linux-on-ibm-z/go.git
    ```

1. Build the bootstrap tool:

    ```
export GOROOT_BOOTSTRAP=$HOME/go1.5.2/go
cd $HOME/go/src
GOOS=linux GOARCH=s390x ./bootstrap.bash
    ```

   The build will place the bootstrap tool in `$HOME/go-linux-s390x-bootstrap`.

## Building the Go tool-chain

To build the Go tool-chain you will need to have successfully built the Go bootstrap tool (see above).

*__NOTE__: If you do not have a home directory that is shared (e.g. via NFS) with the AMD64 machine, then deep copy `$HOME/go-linux-s390x-bootstrap` to `$HOME` on the z Systems machine, and clone the source again:*

```
cd $HOME
git clone http://github.com/linux-on-ibm-z/go.git
```

1. Build the Go tool-chain on z Systems and run all tests.

    ```
export GOROOT_BOOTSTRAP=$HOME/go-linux-s390x-bootstrap
cd $HOME/go/src
./all.bash
    ```

    *__NOTE__: If most of the tests pass, but not quite all, then the Go tool-chain has probably compiled OK and you can proceed to the next step.*

1. Set up your path to use the new tool-chain. Now you can compile Go programs on Linux on z Systems.

    ```
export PATH=$HOME/go/bin:$PATH
    ```