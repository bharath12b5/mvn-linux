The [oCaml 4.02.1 interpreter](https://ocaml.org/releases/4.02.html) can be built on SLES12/RHEL7 on IBM z System by following these instructions.

1. Download latest tarball from [ocaml site](https://ocaml.org/releases/4.02.html)

        wget http://caml.inria.fr/pub/distrib/ocaml-4.02/ocaml-4.02.1.tar.gz
        tar -zxvf ocaml-4.02.1.tar.gz

2. ocaml provided detail INSTALL [instruction]( http://caml.inria.fr/pub/distrib/ocaml-4.02/notes/INSTALL). From top directory e.g. ˜/ocaml-4.02.1. Configure the system.

        ./configure

3. Build oCaml bytecode compiler

        make world

4. Recompile all OCaml sources with the newly created compiler.

        make bootstrap

5. Insall oCaml, from the top directory, become superuser

        umask 022
        make install

6. Verify the build manually 

        ocaml

7. Verify the build by running the provided testsuite, from top directory

        cd testsuite
        make all

[Known problem reported](http://caml.inria.fr/mantis/view.php?id=6630)
