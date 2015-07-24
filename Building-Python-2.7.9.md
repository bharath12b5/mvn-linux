The following build instructions have been tested with **Python 2.7.9** on **RHEL 6.5, 7.0 and SLES 11.3, 12.0 on IBM z Systems**.

#### Section 1: Install the Dependencies
All of the usable dependencies are available on the RHEL 6.5, 7.0 and SLES 11.3, 12.0. In particular,

1. gcc & g++
2. make
3. ncurses
4. patch
5. bzip2-devel

#### Section 2: Build and Install Python 2.7.9
1. Get the source.

        wget https://www.python.org/ftp/python/2.7.9/Python-2.7.9.tar.xz
        tar -xvf Python-2.7.9.tar.xz

2. Configure the build
Skipping this step sill result in installing Python in default location /usr/local.

         cd Python-2.7.9
        ./configure --prefix=<build-location> --exec-prefix=<build-location>

    For instance,

         cd Python-2.7.9
        ./configure --prefix=/usr/local --exec-prefix=/usr/local

3. Build the source

        make

4. (Optional) Run the functional verification test suites.

        make test

5. (Optional) Make verbose test suite.

        ./python Lib/test/regrtest.py -v test_<suite_name>

    For instance,

        ./python Lib/test/regrtest.py -v test_posix

6. Install the binaries.

         make install

#### Section 3: Notes on Verification Test Failures (not specific to Linux on z Systems)
1. `test_posix` and `test_zipimport`will fail, if the tests are run on NFS where the mounted directory is not on a Linux machine.
2. `test_gdb` will crash on SLES 12.0. The crash is due to error in the test source code on line 23 and 24 in file `Python-2.7.9/Lib/test/test_gdb.py`. In particular, with the way the major and minor versions of gdb are extracted.
3. `test_gdb` will fail on RHEL 7.0. The bug has been reported to the community and can be tracked here: [Bug 23137](http://bugs.python.org/issue23137).

#### Section 4: References
1. [Release notes for Python 2.7.9](https://www.python.org/downloads/release/python-279/).