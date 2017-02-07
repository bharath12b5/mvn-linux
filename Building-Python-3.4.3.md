The following build instructions have been tested with **Python 3.4.3** on **RHEL 6.8, 7.1 and SLES 11.4, 12.0 on IBM z Systems**.

#### Section 1: Install the Dependencies
All of the usable dependencies are available on the RHEL 6.8, 7.1 and SLES 11.4, 12.1. In particular,

1. gcc & g++
2. make
3. ncurses
4. patch

#### Section 2: Build and Install Python 3.4.3
1. Get the source.

        wget https://www.python.org/ftp/python/3.4.3/Python-3.4.3.tar.xz
        tar -xvf Python-3.4.3.tar.xz

2. Configure the build
Skipping this step will result in installing Python in default location /usr/local.

         cd Python-3.4.3
        ./configure --prefix=<build-location> --exec-prefix=<build-location>

    For instance,

         cd Python-3.4.3
        ./configure --prefix=/usr/local --exec-prefix=/usr/local


3. Install test patch if installing/testing on NFS

        wget https://bugs.python.org/file38347/test_support.patch
        patch -p1 < test_support.patch

4. Build the source

        make

5. (Optional) Run the functional verification test suites.

        make test

6. (Optional) Make verbose test suite.

         ./python -m test -v test_<suite_name>

    For instance,

        ./python -m test -v test_posix

7. Install the binaries.

         make install

#### Section 3: Notes on Verification Test Failures (not specific to Linux on z Systems)
1. `test_posix` and `test_zipimport`will fail, if the tests are run on NFS where the mounted directory is not on a Linux machine.
2. `test_gdb` will crash on SLES 12.0. The crash is due to error in the test source code on lines 32 and 33 in file `Python-3.4.3/Lib/test/test_gdb.py`. In particular, with the way the major and minor versions of gdb are extracted.
3. There is one failing test case `test_os`, which is already fixed in the higher version. 

#### Section 4: References
1. [Release notes for Python 3.4.3](https://www.python.org/downloads/release/python-343/).

2. [More information about the NFS patch](https://bugs.python.org/issue20876).
