<!---PACKAGE:R--->
<!---DISTRO:SLES 11:3.1.3--->
<!---DISTRO:SLES 12:3.2.2--->
These instructions describe how to build [R](https://www.r-project.org) for IBM Linux on z Systems SLES 11 and SLES 12.

Version 3.1.3 for SLES 11 and version 3.2.2 for SLES 12 has been successfully built this way.

**NOTE:** _When following the steps below, please use a standard permission user unless otherwise specified._

R 3.2.x requires texinfo > 5.1 (which is not supported on SLES versions < 12.2).

## Creating Local Repository
Creating a local repository will ease installation of the dependencies:

        mkdir $HOME/rpms
        cd $HOME/rpms
        sudo zypper addrepo $PWD local

## Installing Build Dependencies on SLES 11


1.  Download the source rpm into the local repository and refresh it:
      
        wget http://download.opensuse.org/repositories/devel:/languages:/R:/patched/SLE_11_SP3/src/R-patched-3.1.3-27.1.src.rpm
        sudo zypper refresh

2. Use zypper to install the dependencies of the source rpm:
    
        sudo zypper source-install -d R-patched

## Installing Build Dependencies on SLES 12

1.  Download the source rpm into the local repository and refresh it:
      
        wget http://download.opensuse.org/repositories/devel:/languages:/R:/patched/SLE_12/src/R-patched-3.2.2-53.1.src.rpm
        wget http://download.opensuse.org/repositories/devel:/languages:/R:/patched/SLE_12/src/perl-Text-Unidecode-1.23-6.8.src.rpm
        wget ftp://fr2.rpmfind.net/linux/fedora-secondary/development/rawhide/source/SRPMS/t/texinfo-5.2-1.fc21.src.rpm
        sudo zypper refresh

2. Install build dependencies, build and install perl-Text-Unidecode:

        sudo zypper source-install -d perl-Text-Unidecode
        rpmbuild --rebuild perl-Text-Unidecode-1.23-6.7.src.rpm
        cp $HOME/rpmbuild/RPMS/noarch/perl-Text-Unidecode-1.23-6.7.noarch.rpm .
        sudo zypper refresh
        sudo zypper install perl-Text-Unidecode

3. Install build dependencies, build and install texinfo:

        sudo zypper source-install -d texinfo
        rpmbuild --rebuild texinfo-5.2-1.fc21.src.rpm
        cp $HOME/rpmbuild/RPMS/*/info-*.rpm \
             $HOME/rpmbuild/RPMS/*/makeinfo-*.rpm \
             $HOME/rpmbuild/RPMS/*/texinfo-*.rpm .
        sudo zypper refresh
        sudo zypper install texinfo

## Building Source

Build the source rpm:

on SLES 11:
   
    rpmbuild —rebuild R-patched-3.1.3-27.1.src.rpm

on SLES 12:
   
    rpmbuild —rebuild R-patched-3.2.2-53.1.src.rpm
    
## Installing R on SLES:

1.  Copy the built rpm to the local repository and refresh.

    On SLES 11:

        cp /usr/src/packages/RPMS/*/R-patched*.rpm .
        sudo zypper refresh

    On SLES 12:

        cp $HOME/rpmbuild/RPMS/*/R-patched*.rpm .
        sudo zypper refresh

2. Install R and its dependencies:
     
        sudo zypper install R-patched

3. On SLES 12, change permissions to R binaries to allow everyone access:

        sudo chmod go=rx -R /usr/lib64/R


## Cleaning up

Remove the repository and delete its files:

    cd $HOME
    sudo zypper removerepo local
    sudo zypper refresh
    rm -Rf rpms
