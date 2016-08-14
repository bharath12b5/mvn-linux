<!---PACKAGE:OCaml--->
<!---DISTRO:SLES 12:4.03--->
<!---DISTRO:SLES 11:4.03--->
<!---DISTRO:RHEL 7.1:4.03--->
<!---DISTRO:RHEL 6.6:4.03--->
<!---DISTRO:Ubuntu 16.x:4.03--->

### Building OCaml

Below version of OCaml is available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `4.02.3`

[OCaml](http://ocaml.org) is a popular object-oriented functional programming language. The OCaml system contains an interpreter as well as a compiler (that compiles OCaml source to machine code). The instructions provided below specify the steps to build OCaml 4.03.0 on Linux on the IBM z Systems for RHEL 6/7, SLES 11/12 and Ubuntu 16.04.  

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._


## Pre-requisites

1. Ensure that GCC and GNU make are installed and up to date:

   (RHEL 6/7)

        sudo yum install gcc make git

   (SLES 11/12)

        sudo zypper install gcc make git
		
	(UBUNTU 16.04)
		
		sudo apt-get update
		sudo apt-get install gcc make git

2. Download the OCaml source and navigate into the top-level directory:

        git clone https://github.com/ocaml/ocaml.git
        cd ocaml
		git checkout 4.03.0

## Building and Installation

1. Configure the build by issuing this command:

        ./configure

2. Build the OCaml bytecode compiler:

        make world

3. Build the native-code compiler, then use it to compile fast versions of the OCaml compilers:

        make opt
        make opt.opt

4. Install the OCaml system:

        umask 022
        sudo make install

# References  
  http://ocaml.org  
  http://caml.inria.fr/ocaml/release.en.html