# Building Python 3.5.1
The following build instructions have been tested with **Python 3.5.1** on **RHEL 6, 7 and SLES 11, 12 on IBM z Systems**.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

#### Section 1: Install the Dependencies
All of the usable dependencies are available on the RHEL 6, 7 and SLES 11, 12. In particular,

* RHEL7

		sudo yum install -y gcc gcc-c++ make ncurses patch wget tar
		
* RHEL6

		sudo yum install -y gcc gcc-c++ make ncurses patch xz xz-devel wget tar
		
* SLES12

		sudo zypper install -y gcc gcc-c++ make ncurses patch wget tar
		
* SLES11

		sudo zypper install -y gcc gcc-c++ make ncurses patch zlib zlib-devel wget tar
		
#### Section 2: Build and Install Python 3.5.1
1. Get the source.

		cd /<source_root>/
        wget https://www.python.org/ftp/python/3.5.1/Python-3.5.1.tar.xz
        tar -xvf Python-3.5.1.tar.xz

2. Configure the build.  Skipping this step will result in installing Python in default location /usr/local.

         cd /<source_root>/Python-3.5.1
        ./configure --prefix=<build-location> --exec-prefix=<build-location>

    For instance,

         cd /<source_root>/Python-3.5.1
        ./configure --prefix=/usr/local --exec-prefix=/usr/local

3. Build the source.

        make

4. **(Optional)** Run the functional verification test suites.

        make test

5. **(Optional)** Make verbose test suite.

         ./python -m test -v test_<suite_name>

    For instance,

        ./python -m test -v test_posix

6. Install the binaries.

         sudo make install


#### Section 3: References
1. [Release notes for Python 3.5.1](https://www.python.org/downloads/release/python-351/)
