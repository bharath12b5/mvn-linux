[OCaml](http://ocaml.org) is a popular object-oriented functional programming language. The OCaml system contains an interpreter as well as a compiler (that compiles OCaml source to machine code). The latest revision of the code now support Linux on z Systems. The following instructions describe how to build OCaml on RHEL and SLES.

**NOTE**: When following the steps below please use a standard permission user unless otherwise specified.

## Pre-requisites

1. Ensure that GCC and GNU make are installed and up to date:

   (RHEL 6/7)

        sudo yum install gcc make

   (SLES 11/12)

        sudo zypper install gcc make

2. Download the OCaml source and navigate into the top-level directory:

        git clone https://github.com/ocaml/ocaml.git
        cd ocaml

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