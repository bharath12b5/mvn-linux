<!---PACKAGE:Doxygen--->
<!---DISTRO:SLES 12.x:Distro, 1.8--->
<!---DISTRO:SLES 11.x:Distro, 1.8--->
<!---DISTRO:RHEL 7.x:Distro, 1.8--->
<!---DISTRO:RHEL 6.x:Distro, 1.8--->
<!---DISTRO:Ubuntu 16.x:Distro, 1.8--->

# Building Doxygen

Below versions of Doxygen are available in respective distributions at the time of this recipe creation:

* RHEL 7.1/7.2/7.3 have  `1.8.5` 
* RHEL 6.8 has  `1.6.1` 
* SLES 12/12-SP1/12-SP2 have  `1.8.6-1.20` 
* SLES 11-SP4 has  `1.5.6-1.19` 
* Ubuntu 16.04/16.10 have `1.8.11-1` 

The instructions provided below specify the steps to build Doxygen version 1.8.13 on the IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_
      
* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##Step 1: Building and installing Doxygen

####1.1) Install dependencies

  * RHEL 6.8
    ```sh
    sudo yum install -y git flex bison gcc gcc-c++ texlive-latex.s390x make python-argparse.noarch openssl wget tar qt-devel-4.6.2-28.el6_5.s390x
    ```

  * RHEL 7.1/7.2/7.3
    ```sh
    sudo yum install -y git flex bison gcc gcc-c++ texlive-bibtex-bin make perl-Test-Simple.noarch openssl wget tar cpan qt-devel latex ghostscript texlive unzip texlive-epstopdf.noarch texlive.s390x python-argparse.noarch
    ```

  * SLES 11-SP4
    ```sh
    sudo zypper install -y git flex bison gcc gcc-c++ make openssl wget tar libqt4-devel libxml2-devel texlive ghostscript-devel texlive-bin-tools unzip graphviz python-xml python-argparse libxml2-devel perl perl-YAML zip texlive*
    ```

  * SLES 12/12-SP1/12-SP2
    ```sh
    sudo zypper install -y git flex bison gcc gcc-c++ python-xml libxml2-tools libxml2-devel texlive-bibtex-bin make perl-Test-Simple.noarch  openssl wget tar libqt4-devel ghostscript texlive unzip graphviz perl perl-YAML zip texlive*
    ```
 
  * Ubuntu 16.04/16.10
    ```sh
    sudo apt-get update
    sudo apt-get install -y git cmake python flex bison qt-sdk texlive libc6 libclang1-3.* libgcc1 libstdc++6 doxygen-doc doxygen-gui doxygen-latex graphviz libxml2-utils
    ```

####1.2) Create a working directory to use as a Doxygen installation workspace  
  ```sh
  mkdir /<source_root>/
  cd /<source_root>/
  ```

####1.3) Build CMAKE (Only for RHEL/SLES)
  ```sh
  wget http://www.cmake.org/files/v2.8/cmake-2.8.12.1.tar.gz
  gzip -d cmake-2.8.12.1.tar.gz
  tar xvf cmake-2.8.12.1.tar
  cd cmake-2.8.12.1
  ./configure --prefix=/cmake-2.8.12.1/cmake
  make
  sudo make install
  export PATH=/<source_root>/cmake-2.8.12.1/bin:$PATH
  ```
 
####1.4) Use CPAN to install Perl modules (Only for RHEL 7, SLES 11, SLES 12)
  ```sh	
  sudo cpan Test::More
  sudo cpan File::Path
  ```
  **_Note_**: 
  * _If CPAN has not previously been setup it asks setup/configuration questions before the requested Perl module is installed. The answers can be site specific, but as a guideline, it is generally OK to accept the defaults or suggestions offered._
  * _If Perl fails to build, it may be because 'echo port 7 ' is commented out in the services file. Please check in the /etc/services file, that the following lines exist and are active (e.g. Not preceded by a # symbol to comment them out)._

####1.5) Install dependencies(latex) for make docs command: (for RHEL/SLES)
  ```sh
  wget http://mirrors.ctan.org/macros/latex/contrib/multirow/multirow.dtx
  wget http://mirrors.ctan.org/macros/latex/contrib/multirow/multirow.ins
  wget http://mirrors.ctan.org/macros/latex/contrib/multirow/multirow.pdf
  tex multirow.ins
	
  wget  http://mirrors.ctan.org/macros/latex/contrib/import/import.sty 
  wget http://mirrors.ctan.org/macros/latex/contrib/xtab/xtab.dtx
  wget http://mirrors.ctan.org/macros/latex/contrib/xtab/xtab.ins 
  wget http://mirrors.ctan.org/macros/latex/contrib/xtab/xtab.pdf 
  latex xtab.ins 
	
  wget http://mirrors.ctan.org/macros/latex/contrib/sectsty/sectsty.dtx
  wget http://mirrors.ctan.org/macros/latex/contrib/sectsty/sectsty.ins 
  wget http://mirrors.ctan.org/macros/latex/contrib/sectsty/sectsty.pdf 
  latex sectsty.ins 
	
  wget http://mirrors.ctan.org/macros/latex/contrib/tocloft/tocloft.dtx 
  wget http://mirrors.ctan.org/macros/latex/contrib/tocloft/tocloft.ins 
  wget http://mirrors.ctan.org/macros/latex/contrib/tocloft/tocloft.pdf 
  latex tocloft.ins 
	
  wget http://mirrors.ctan.org/macros/latex/contrib/appendix/appendix.dtx 
  wget http://mirrors.ctan.org/macros/latex/contrib/appendix/appendix.ins 
  wget http://mirrors.ctan.org/macros/latex/contrib/appendix/appendix.pdf 
  latex appendix.ins 
	
  wget http://mirrors.ctan.org/macros/latex/contrib/tabu/tabu.dtx 
  wget http://mirrors.ctan.org/macros/latex/contrib/tabu/tabu.ins 
  wget http://mirrors.ctan.org/macros/latex/contrib/tabu/tabu.pdf 
  latex tabu.ins
  ```
	
  *  Perform the following steps for SLES 12/12-SP1/12-SP2
  ```  sh
  wget http://mirrors.ctan.org/support/epstopdf/epstopdf.pl  
  sudo mv epstopdf.pl /usr/local/bin/epstopdf  
  chmod a+x /usr/local/bin/epstopdf
  export PATH=/usr/local/bin:$PATH
  ```      
 
  *  The files are made available to latex by placing at specific location
  ```sh
  sudo mkdir -p /usr/share/texmf/tex/latex/ (Only for RHEL 7)
  sudo cp *.sty /usr/share/texmf/tex/latex/  
  sudo mktexlsr
  ```

####1.6) Check-out Doxygen from github, it will be placed in a sub-directory `doxygen`
  ```sh
  cd /<source_root>/
  git clone https://github.com/doxygen/doxygen.git
  cd doxygen
  git checkout Release_1_8_13
  ```

####1.7) Apply changes to the following file (**Only** for Ubuntu)
 * `/<source_root>/doxygen/doc/translator.py`
   ```diff
    --- a/doc/translator.py
    +++ b/doc/translator.py
    @@ -83,7 +83,7 @@ def xopen(fname, mode='r', encoding='utf-8-sig'):
         the default 'utf-8-sig' is used (skips the BOM automatically).
         '''

    -    major, minor, patch = (int(e) for e in platform.python_version_tuple())
    +    major, minor, patch = sys.version_info[:3]
         if major == 2:
             return open(fname, mode=mode) # Python 2 without encoding
         else:
    @@ -1990,7 +1990,7 @@ class TrManager:
     if __name__ == '__main__':

         # The Python 2.6+ or 3.3+ is required.
    -    major, minor, patch = (int(e) for e in platform.python_version_tuple())
    +    major, minor, patch = sys.version_info[:3]
         if (major == 2 and minor < 6) or (major == 3 and minor < 0):
             print('Python 2.6+ or Python 3.0+ are required for the script')
             sys.exit(1)
   ```

####1.8) Create a build directory where the output should be stored
  ```sh
  cd /<source_root>/doxygen
  mkdir build
  cd build
  ```

####1.9) Build Doxygen using cmake
  ```sh
  cd /<source_root>/doxygen/build
  cmake -G "Unix Makefiles" -Dbuild_doc=ON -Dbuild_wizard=YES ..
  make  
  make docs  
  sudo make install 
  ```
	
##Step 2: Run the functional test suite (Optional)
  ```sh
  cd /<source_root>/doxygen/build
  make tests
  ```  

### References:
https://github.com/doxygen/doxygen
