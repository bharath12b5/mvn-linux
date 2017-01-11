# Building PostgreSQL

Below versions of PostgreSQL are available in respective distributions at the time of this recipe creation:

* RHEL 6.8 has  `8.4.20`
* RHEL 7.1 & 7.2 have `9.2.15`
* RHEL 7.3 has `9.2.18`
* SLES 11-SP4 has 8.3.5
* Ubuntu 16.04 has  `9.5+173`
* Ubuntu 16.10 has  `9.5+176`

The instructions provided below specify the steps to build PostgreSQL version 9.6.1 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.


_**General Notes:**_
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##Step 1: Building and installing PostgreSQL
####1.1) Install dependencies

* RHEL 6.8 and RHEL 7.1/7.2/7.3
  ```bash
  sudo yum install git wget build-essential gcc gcc-c++ make readline-devel zlib-devel bison flex
  ```

* SLES 11-SP4
  ```bash
  sudo zypper install git gcc  gcc-c++ make readline-devel  zlib-devel bison flex awk
  ```

* SLES 12/12-SP1/12-SP2
  ```bash
  sudo zypper install git gcc  gcc-c++ make readline-devel  zlib-devel bison flex
  ```

* Ubuntu 16.04/16.10
  ```bash
  sudo apt-get update
  sudo apt-get install bison flex wget build-essential git gcc make zlib1g-dev libreadline6 libreadline6-dev
  ```

####1.2) Create postgres user, group and home directory

  ```bash
  sudo groupadd -r postgres
  sudo useradd -r -m -g postgres postgres
  sudo passwd postgres
  ```
 
  _**Note:** Password for SLES11-SP4 should be up to 7 characters. Please note that `/usr/sbin` is available in `PATH` environment variable._

####1.3) Download PostgreSQL source code from github

  ```bash
  su postgres -s /bin/bash
  cd /<source_root>/
  git clone https://github.com/postgres/postgres.git
  cd postgres/
  git checkout REL9_6_1
  ```

  _**Note:** Login as postgres user and download the source in postgres home directory._

####1.4) Build and install PostgreSQL

  ```bash
  cd /<source_root>/postgres
  ./configure
  make
  make check
  sudo make install
  ```
 _**Note:** Before you run `make check` make sure LANG environment variable is *not* set. `unset LANG` if it is already set._

####1.5) Update the PATH variable

  ```bash
  export PATH=$PATH:/usr/local/pgsql/bin
  ``` 

##Step 2: Set up PostgreSQL server (Optional)
####2.1) Create PostgreSQL data directory to store data and make postgres user as the owner

  ```bash
  sudo mkdir /usr/local/pgsql/data
  sudo chown postgres:postgres /usr/local/pgsql/data
  ```

####2.2) Initialize PostgreSQL data directory as postgres user

  ```bash
  su postgres -s /bin/bash
  /usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data/
  ```	

  _**Note:** Please make sure the directory `/usr/local/` has sufficient read and execute permissions when initializing._

####2.3) Start the PostgreSQL server

  ```bash
  /usr/local/pgsql/bin/postmaster -D /usr/local/pgsql/data
  ```

## References:
https://www.postgresql.org/   
https://github.com/postgres/postgres.git
