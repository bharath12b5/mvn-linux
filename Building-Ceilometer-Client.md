The following build instructions have been tested with Ceilometer Client 1.0.13 on RHEL 7 & 6 and SLES 12 & 11 for IBM Linux on z Systems.

_**General Notes:**_ 	

i) When following the steps below please use a standard permission user unless otherwise specified.
	 
ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

## Building Ceilometer Client

1. Install the build system dependencies

    For RHEL 7.1 & 6.6
    ```shell
    sudo yum install git python-devel python-setuptools gcc findutils
    ```
    For SLES 12 & 11
    ```shell
    sudo zypper install git python-devel python-setuptools gcc findutils
    ```
2. Install python modules dependencies

    For RHEL 7.1 & 6.6
    ```shell
    sudo easy_install pip
    ```
    For SLES 12 & 11
    ```shell
    sudo easy_install pip==1.2.1
    ```
    Now use pip to install other dependencies
    ```shell
    sudo pip install virtualenv
    sudo pip install pbr
    ```
3. Download the required version of Ceilometer

    ```shell
    cd /<source_root>/
    git clone https://github.com/openstack/python-ceilometerclient.git
    cd python-ceilometerclient
    git checkout 1.0.13
    ```
4. Install Ceilometer's python requirements

    ```shell
    sudo pip install -r requirements.txt
    ```
5. (_Optional_) Install test requirements and test

    SLES 11 Only
    ```shell
    sudo pip install linecache2
    sudo pip install unittest2
    ```
    _**Note:** pip 1.2.1 install on SLES 11 sometimes has issues - if one of these fails to install, simply run the install a second time and it will complete_  
	
    For all platforms
    ```shell
    sudo pip install -r test-requirements.txt
    ./run_tests.sh -N
    ```
    _**Note:** SLES 12 reports a syntax error whilst installing unittest2. However, the Client successfully installs and passes all 192 tests_
	
6. Install and verify

    ```shell
    sudo python setup.py install
    ceilometer --help
    ```

##References:
https://github.com/openstack/python-ceilometerclient/