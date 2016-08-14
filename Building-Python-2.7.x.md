Below versions of Python are available in respective distributions at the time of this recipe creation:

*    RHEL 6.6 has `2.6.6`
*    RHEL 7.1 has `2.7.5`
*    SLES 11 has `2.6.9`
*    SLES 12 has `2.7.9`
*    Ubuntu 16.04 has `2.7.11`

The instructions provided below specify the steps to build Python 2.7.12 on Linux on the IBM z Systems for RHEL 6/7, SLES 11/12 and Ubuntu 16.04.

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

* Ubuntu 16.04
		
		sudo apt-get update
		sudo apt-get install -y gcc g++ make libncurses5-dev libreadline6-dev libssl-dev libgdbm-dev libc6-dev libsqlite3-dev libbz2-dev xz-utils patch wget tar curl patch

#### Section 2: Build and Install Python 2.7.12
1. Get the source

        wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tar.xz
        tar -xvf Python-2.7.12.tar.xz

2. Configure the build 

	Skipping this step will result in installing Python in default location /usr/local.

         cd Python-2.7.12
        ./configure --prefix=<build-location> --exec-prefix=<build-location>

    For instance,

         cd Python-2.7.12
        ./configure --prefix=/usr/local --exec-prefix=/usr/local

3. Build the source

        make

4. (Optional) Run the functional verification test suites

        make test

	_**Note:** In order to disable SSLv3 in OpenSSL without breaking ABI, Ubuntu 16.04 LTS will ship with OP_NO_SSLv3 forced on by default. This is causing the `test_ssl.py` test to fail. Below is a patch to fix the issue._

		curl http://bugs.python.org/file41150/fix-sslv3-test.diff | patch -p1

5. (Optional) Make verbose test suite

        ./python Lib/test/regrtest.py -v test_<suite_name>

    For instance,

        ./python Lib/test/regrtest.py -v test_posix

6. Install the binaries

         sudo make install


#### Section 3: References
1. [Release notes for Python 2.7.12](https://www.python.org/downloads/release/python-2712/).