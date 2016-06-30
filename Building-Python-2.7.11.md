Below versions of Python are available in respective distributions at the time of this recipe creation:

*    RHEL 6.6 has `2.6.6`
*    RHEL 7.1 has `2.7.5`
*    SLES 11 has `2.6.9`
*    SLES 12 has `2.7.9`
*    Ubuntu 16.04 has `2.7.11`

The following build instructions have been tested with **Python 2.7.11** on **RHEL 6.6, 7.1 and SLES 11, 12 on IBM z Systems**.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

#### Section 1: Install the Dependencies

* RHEL7.1

		sudo yum install -y gcc gcc-c++ make ncurses patch wget tar
		
* RHEL6.6

		sudo yum install -y gcc gcc-c++ make ncurses patch xz xz-devel wget tar
		
* SLES12

		sudo zypper install -y gcc gcc-c++ make ncurses patch wget tar
		
* SLES11

		sudo zypper install -y gcc gcc-c++ make ncurses patch zlib zlib-devel wget tar

#### Section 2: Build and Install Python 2.7.11
1. Get the source

        wget https://www.python.org/ftp/python/2.7.11/Python-2.7.11.tar.xz
        tar -xvf Python-2.7.11.tar.xz

2. Configure the build 

	Skipping this step will result in installing Python in default location /usr/local.

         cd Python-2.7.11
        ./configure --prefix=<build-location> --exec-prefix=<build-location>

    For instance,

         cd Python-2.7.11
        ./configure --prefix=/usr/local --exec-prefix=/usr/local

3. Build the source

        make

4. (Optional) Run the functional verification test suites

        make test

5. (Optional) Make verbose test suite

        ./python Lib/test/regrtest.py -v test_<suite_name>

    For instance,

        ./python Lib/test/regrtest.py -v test_posix

6. Install the binaries

         sudo make install


#### Section 3: References
1. [Release notes for Python 2.7.11](https://www.python.org/downloads/release/python-2711/).