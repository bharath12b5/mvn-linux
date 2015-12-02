# Building Doxygen

Doxygen can be built for Linux on z Systems running RHEL 6.6, RHEL 7.1, SLES 11 and SLES 12, by following these instructions. Version 1.8.10 has been successfully built & tested this way.

##### General Notes:
      
i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it.

### Building Doxygen

1. Install standard utilities, packages and platform specific dependencies :

 RHEL7.1:
 ```
      yum install -y git flex bison gcc gcc-c++ texlive-bibtex-bin make perl-Test-Simple.noarch openssl wget tar cpan qt-devel-4.8.5-8.el7.s390x latex ghostscript texlive unzip texlive-epstopdf.noarch texlive.s390x 
 ```

 RHEL6.6:
 ```
      yum install -y git flex bison gcc gcc-c++ texlive-bibtex-bin make perl-Test-Simple.noarch openssl wget tar cpan qt-devel-4.8.5-8.el7.s390x texlive-latex.s390x  ghostscript texlive unzip texlive-epstopdf.noarch texlive.s390x 
 ```

 SLES12:
 ```
      zypper install -y git flex bison gcc gcc-c++ python-xml libxml2-tools libxml2-devel texlive-bibtex-bin make perl-Test-Simple.noarch  openssl wget tar libqt4-devel latex ghostscript texlive unzip graphviz perl perl-YAML zip texlive*
 ```
 
 SLES11:
 ```
      zypper install -y git flex bison gcc gcc-c++ make openssl wget tar libqt4-devel libxml2-devel texlive ghostscript-devel texlive-bin-tools unzip graphviz python-xml libxml2-devel perl perl-YAML zip texlive*
 ```

2. Use CPAN to install Perl modules:
   ```	
      export PATH=$PATH:/root/perl5/bin
      export PERL5LIB=$PERL5LIB:/root/perl5/lib/perl5
      export PERL_LOCAL_LIB_ROOT=$PERL_LOCAL_LIB_ROOT:/root/perl5
      export PERL_MB_OPT=$PERL_MB_OPT:"--install_base \"/root/perl5\""
      export PERL_MM_OPT=$PERL_MM_OPT:"INSTALL_BASE=/root/perl5"
      cpan Test::More
      cpan File::Path
   ```
   **_Note_**: If CPAN has not previously been setup it asks setup/configuration questions before the requested Perl module is installed. The answers can be site specific, but as a guideline, it is generally OK to accept the defaults or suggestions offered.  
   **_Note_**: If Perl fails to build, it may be because 'echo port 7 ' is commented out in the services file. Please check in the /etc/services file, that the following lines exist and are active (e.g. Not preceeded by a # symbol to comment them out).

3. Install dependencies(latex) for make docs command

 ```
      wget http://mirrors.ctan.org/macros/latex/contrib/multirow/multirow.sty
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
 *  Perform the following steps for SLES12  

   ```  
      wget http://mirrors.ctan.org/support/epstopdf/epstopdf.pl  
      mv epstopdf.pl /usr/local/bin/epstopdf  
      chmod a+x /usr/local/bin/epstopdf
   ```      
 *  The files are made available to latex by placing at specific location  

   ```  
      cp *.sty /usr/share/texmf/tex/latex/  
      mktexlsr
   ```

4. Create a working directory with write permission to use as a Doxygen installation workspace  
(Referred to as /\<source_root\>/ from this point on) :

   ```
      mkdir /<source_root>/
      cd /<source_root>/
   ```

5. Check-out Doxygen from github, it will be placed in a sub-directory doxygen : 
      ```
      git clone https://github.com/doxygen/doxygen.git/
      cd doxygen
      git checkout Release_1_8_10
      ```
   **_Note_**: It is not necessary to checkout a specific version. The example below downloads 1.8.10 version, that was current when this recipe was created.  
  

6. Create a build directory where the output should be stored
   ```
        cd doxygen
        mkdir build
        cd build
   ```

7. Build CMAKE_2.8.12.1
   ```
      wget http://www.cmake.org/files/v2.8/cmake-2.8.12.1.tar.gz
      gzip -d cmake-2.8.12.1.tar.gz
      tar xvf cmake-2.8.12.1.tar
      cd cmake-2.8.12.1
      ./configure --prefix=/cmake-2.8.12.1/cmake
      make  
      make install  
   ```
8. Build and install Doxygen

   ```
      cd ..
      cmake-2.8.12.1/bin/cmake -G "Unix Makefiles" -Dbuild_doc=ON -Dbuild_wizard=YES .. 
      make  
      make docs  
      make install 
   ```
9. Run the functional test suite and ensure all test cases pass

   ```
      make tests
   ```  

### References:
https://github.com/doxygen/doxygen  
  
  

> 	The information provided in this article is accurate at the time of writing, but on-going development in the 
> 	open-source projects involved may make the information incorrect or obsolete. Please contact us on [developerWorks](https://www.ibm.com/developerworks/community/groups/community/lozopensource)
> 	if you have any questions or feedback.