
These instructions describe how to build [R](https://www.r-project.org) for IBM Linux on z Systems.

Version 3.1.3 for SLES 11 has been successfully built this way.

**NOTE:** _When following the steps below, please use a standard permission user unless otherwise specified._

## Creating Local Repository
Creating a local repository will ease installation of the dependencies:

        mkdir $HOME/rpms
        cd $HOME/rpms
        sudo zypper addrepo $PWD local

## Installing Build Dependencies


1. Download the source rpm into the local repository and refresh it:
      
        wget http://download.opensuse.org/repositories/devel:/languages:/R:/patched/SLE_11_SP3/src/R-patched-3.1.3-27.1.src.rpm
        sudo zypper refresh

2. Use zypper to install the dependencies of the source rpm:
    
        sudo zypper source-install -d R-patched

## Building Source

Build the source rpm:

    rpmbuild —rebuild R-patched-3.1.3-27.1.src.rpm

    
## Installing R on SLES:

1.  Copy the built rpm to the local repository and refresh.

        cp /usr/src/packages/RPMS/*/R-patched*.rpm .
        sudo zypper refresh

2. Install R and its dependencies:
     
        sudo zypper install R-patched


## Cleaning up

Remove the repository and delete its files:

    cd $HOME
    sudo zypper removerepo local
    sudo zypper refresh
    rm -Rf rpms
