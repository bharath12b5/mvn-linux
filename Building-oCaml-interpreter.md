The [oCaml 4.02 interpreter](https://ocaml.org/releases/4.02.html) can be built on RHEL 7.1/6.6 or SLES12/11 on IBM z System by following these instructions.

_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified._

1. Ensure you have the necessary dependencies in place  

  RHEL 7.1 & 6.6
  ```Shell
  sudo yum install wget tar gcc make
  ```
  SLES 12 & 11
  ```Shell
  sudo zypper install wget tar gcc make
  ```
1. Download the required tarball from [ocaml site](https://ocaml.org/releases/4.02.html)

  ```Shell
  wget http://caml.inria.fr/pub/distrib/ocaml-4.02/ocaml-4.02.1.tar.gz
  tar -zxvf ocaml-4.02.1.tar.gz
  ```
2. Move to the expanded ocaml-4.02.x directory and configure the system.

  ```Shell
  cd ocaml-4.02.1
  ./configure
  ```
  _**Note:** oCaml provides detailed [install instructions]( http://caml.inria.fr/pub/distrib/ocaml-4.02/notes/INSTALL), these steps are a reduced version of them._

3. Build the oCaml bytecode compiler

  ```shell
  make world
  ```

4. Optional: Recompile all oCaml sources with the newly created compiler - this compares the newly created oCaml bytecode compiler against reference bytecode so you must check that bootstrap passes

  ```shell
  make bootstrap
  ```
  _If bootstrap passes you'll see the message `Fixpoint reached, bootstrap succeeded.`_

5. Switch to root (or other superuser), move to the expanded ocaml-4.02.x directory, and install

  ```shell
  cd ocaml-4.02.1
  umask 022
  make install
  ```

6. Exit the root user (or other superuser), verify the build manually 

  ```shell
  ocaml
  ```
  _**Note:** to quit the oCaml interpreter type `exit 0;;`_

7. Verify the build by running the provided testsuite, from top directory

  ```Shell
  cd testsuite
  make all
  ```
  [One known test problem reported - two files fail](http://caml.inria.fr/mantis/view.php?id=6630)
