# Installing Go 1.7 on Linux on z

These instructions describe how to install version 1.7 of Go for IBM Linux on z Systems running SLES 12, RHEL 7.x or Ubuntu 16.04. These instructions do not require you to download an x86 binary or to cross compile from an x86 machine. All the steps are completed on a Linux on Z server.

Installing Go 1.7 for IBM Linux on z Systems simply involves downloading and unpacking a binary tarball, and possibly one or two other small steps:

## To install Go 1.7 for everyone's use on a system

In a directory of your choice:
   ```
wget https://storage.googleapis.com/golang/go1.7.1.linux-s390x.tar.gz
chmod ugo+r go1.7.1.linux-s390x.tar.gz
sudo tar -C /usr/local -xzf go1.7.1.linux-s390x.tar.gz
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

## To install Go 1.7 for your private use

In a directory of your choice, for example $HOME:
   ```
wget https://storage.googleapis.com/golang/go1.7.1.linux-s390x.tar.gz
tar -xzf go1.7.1.linux-s390x.tar.gz
   ```
Add ``$HOME/go/bin`` to the ``PATH`` environment variable. You can do this by adding this line to your .profile file:

   ```
export PATH=$PATH:$HOME/go/bin
   ```

Redirect the ``GOROOT`` environment variable to ``$HOME/go`` from ``/usr/local/go`` by doing this:

   ```
export GOROOT=$HOME/go
   ```

Also add this to your .profile file to enable CGO:

   ```
export CC=gcc
   ```

## To verify the go installation

Simply run `go version`, it should return

   ```
go version go1.7.1 linux/s390x
   ```
