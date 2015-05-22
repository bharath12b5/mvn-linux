# Building Python Ceilometer Client
The following build instructions have been tested with Ceilometer Client 1.0.13 on RHEL7 and SLES12 on IBM z Systems.

### Version
1.0.13

###Section 1: Install the following Dependencies
*       git (or git-core)
*       python-devel
*       python-setuptools
*       pip
*       python-virtualenv
*       gcc
*       findutils
*       pbr

RHEL7:
```
- yum install -y git \
    python-devel \
    python-setuptools \
    gcc \
    findutils

- easy_install pip
- pip install virtualenv
- pip install pbr
```

SLES12:
```
- zypper install -y git-core \
    python-devel \
    python-setuptools \
    gcc \
    findutils

- easy_install pip==1.2.1
- pip install virtualenv
- pip install pbr
```
####Section 2: Build and Install
1. Re-install the ca certificates for openssl framework (Only for Rhel)
        yum reinstall -y ca-certificates

2.      Get the source from git and checkout v1.0.13.
```sh
   git clone https://github.com/openstack/python-ceilometerclient.git
   cd python-ceilometerclient
   git checkout 1.0.13
```
3.      Install the build-dependencies using Pip
```sh
   pip install -r requirements.txt
   pip install -r test-requirements.txt
```
4.      Run the test suites.
```sh
       ./run_tests.sh -N
```
5.      Install the binaries.
```sh
      python setup.py install
```

###Verification:
To verify run `ceilometer --help`. It should display the options and Usage of ceilometer.

##References:
https://github.com/openstack/python-ceilometerclient/

[Ceilometer]:https://github.com/openstack/python-ceilometerclient.git