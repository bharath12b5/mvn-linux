# Building Go  

Below versions of Go are available in respective distributions at the time of this recipe creation:   

* Ubuntu 16.04/16.10 have `1.6`

The instructions provided below specify the steps to build Go version 1.7.4 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10  

_**General Notes:**_   
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._


##Step 1: Building and Installing Go 
####1.1) Install dependencies  

   * RHEL 6.8 & RHEL 7.1/7.2/7.3  

		sudo yum install tar wget gcc

   * SLES 11-SP4  & SLES 12/12-SP1/12-SP2  

		sudo zypper install tar wget gcc
   
   * Ubuntu 16.04/16.10  

		sudo apt-get update
		sudo apt-get install wget tar

####1.2) To install Go 1.7.4 for everyone's use on a system

In a directory of your choice:
   ```
cd /<source_root>/
wget https://storage.googleapis.com/golang/go1.7.4.linux-s390x.tar.gz
chmod ugo+r go1.7.4.linux-s390x.tar.gz
sudo tar -C /usr/local -xzf go1.7.4.linux-s390x.tar.gz
   ```
Add ``/usr/local/go/bin`` to the ``PATH`` environment variable. You can do this system-wide by adding this line to /etc/profile:

   ```
export PATH=$PATH:/usr/local/go/bin
   ```

On SLES or RHEL perfom this extra step. ``cd`` to the directory where gcc is installed, usually ``/usr/bin``, then execute

   ```
sudo ln gcc s390x-linux-gnu-gcc

   ```
This step is not necessary on Ubuntu as the link is already created.

#### 1.3) To install Go 1.7.4 for your private use

In a directory of your choice, for example $HOME:
   ```
cd /<source_root>/
wget https://storage.googleapis.com/golang/go1.7.4.linux-s390x.tar.gz
tar -xzf go1.7.4.linux-s390x.tar.gz
   ```
Add ``$HOME/go/bin`` to the ``PATH`` environment variable. You can do this by adding this line to your .profile file:

   ```
export PATH=$PATH:$HOME/go/bin
   ```

Redirect the ``GOROOT`` environment variable to ``/<source_root>/go`` from ``/usr/local/go`` by doing this:

   ```
export GOROOT=/<source_root>/go
   ```

Also add this to your .profile file to enable CGO:

   ```
export CC=gcc
   ```

## Step 2: Testing (Optional)
####2.1)  Command `go version` should output

    go version go1.7.4 linux/s390x

### References:
https://golang.org/
