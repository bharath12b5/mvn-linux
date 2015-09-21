Doxygen can be built for Linux on z Systems running RHEL 6.6, RHEL 7.1, SLES 11 and SLES 12 by following these instructions.  Version 1.8.9.1 has been successfully built & tested this way.

_**General Notes:**_ 	

i) When following the steps below please use a standard permission user unless otherwise specified.
	 
ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it


## Building Doxygen

1. Install standard utilities, packages and platform specific dependencies :

    RHEL 6.6
   ```shell
   sudo yum install git bison flex make gcc gcc-c++ openssl graphviz texlive  perl cpan perl-YAML wget tar zip
   ```
    RHEL 7.1
   ```shell
   sudo yum install git bison flex make gcc gcc-c++ openssl graphviz texlive-bibtex-bin  perl cpan perl-YAML wget tar zip
   ```
    SLES 11
    ```shell
   sudo zypper install git bison flex make gcc gcc-c++ openssl graphviz texlive texlive-latex python-xml  perl perl-YAML wget tar zip
   ```
    SLES 12
    ```shell
   sudo zypper install git bison flex make gcc gcc-c++ openssl graphviz texlive-bibtex-bin python-xml libxml2-tools perl perl-YAML wget tar zip
   ```

 _**Note:**_ bibtex is included in package texlive-bibtex-bin (RHEL7.1 and SLES12),  texlive-latex on SLES11 and texlive on RHEL6.6. If this is NOT installed, Test 12 - '[012_cite.dox]: test the \cite command' - (See step 6 below) - will fail.

2. Use CPAN to install Perl modules :

   ```shell
   sudo cpan Test::More
   sudo cpan File::Path
   ```

 _**Note:**_ If CPAN has not previously been setup it asks setup/configuration questions before the requested Perl module is installed. The answers can be site specific, but as a guideline, it is generally OK to accept the defaults or suggestions offered.

3. Create a working directory with write permission to use as a `Doxygen` installation workspace (Referred to as `/<source_root>/` from this point on) :

   ```shell
   mkdir /<source_root>/
   cd /<source_root>/
   ```

4. Check-out Doxygen from github, it will be placed in a sub-directory `doxygen` :

	_**Note:**_ It is not necessary to checkout a specifc version. The example below downloads commit `2e39e5c7c1427ac6b24c64b7ef01be8d5a20092b` that was current when this recipe was created.

   ```shell
   git clone https://github.com/doxygen/doxygen
   cd doxygen
   git  checkout 2e39
   ```

5. Configure and build Doxygen

 _**Note:**_ The example below sets a prefix that includes the version number.  This is optional.

    ```shell
    ./configure --prefix=/opt/doxygen_1.891
    make
    ```
6. Run the supplied Doxygen tests, and verify that all 65 tests PASS

    ```shell
    make test
    ```

7. As root, install the build :

    ```shell
    sudo make install
    ```

8. **OPTIONAL** Change directory, and if desired, delete  `<source_root>` :
    ```shell
    cd /<home>/
    sudo rm -rf /<source_root>/
    ```

