<!---PACKAGE:Ansible--->
<!---DISTRO:RHEL 6.6:2.1.x--->
<!---DISTRO:RHEL 7.1:2.1.x--->
<!---DISTRO:SLES 11:2.1.x--->
<!---DISTRO:SLES 12:2.1.x--->
<!---DISTRO:Ubuntu 16.x:2.1.x--->

## Building Ansible on RHEL 6.6, RHEL 7.1, SLES 11, SLES 12 and Ubuntu 16.04

Below versions of Ansible are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `	2.0.0.2-2`

Latest Ubuntu builds are available [in a PPA here](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-apt-ubuntu).

The instructions provided below specify the steps to build [Ansible](http://www.ansible.com/) version 2.1.0.0 on Linux on the IBM z Systems for RHEL 6.6, RHEL 7.1, SLES 11, SLES 12.


_**General Notes:**_ 	 
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

1. Install dependencies:

    On RHEL 6.6 and RHEL 7.1:
  
        sudo yum install -y git wget tar svn bzip2 unzip gcc python-devel make which autoconf net-tools ssh python-setuptools python-lxml python-ldap sqlite-devel openldap-devel libxslt-devel openssl-devel libffi-devel openssl
      
    On SLES 11:

         sudo zypper install -y git tar bzip2 unzip python-devel make autoconf net-tools wget python-setuptools python-lxml python-ldap libxslt-devel gcc libffi-devel openssl libopenssl-devel mercurial

    On SLES 12:

         sudo zypper install -y git tar bzip2 unzip python-devel make which autoconf net-tools wget python-setuptools python-lxml python-ldap libxslt-devel gcc openssl libopenssl-devel libffi-devel 

2. Build Openssl (**only** on SLES 11)
	
		cd /<source_root>/
		git clone https://github.com/openssl/openssl.git
		cd /<source_root>/openssl
		git checkout OpenSSL_1_0_2d
		./config --prefix=/usr --openssldir=/usr/local/openssl shared
        make
        sudo make install

3. Build Python (**only** on SLES 11)

	The Python version available on the SLES 11 package repositories is too low level so build and install Python yourself following the instructions [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Python-3.5.1).

	Make sure that Python 3.5 is used. Update /usr/bin/python link.

		sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
        sudo update-alternatives --install /usr/bin/python python /usr/local/bin/python3.5 2
        sudo update-alternatives --list python
        sudo  update-alternatives --config python  (Select the option pointing to python3.5)
             

4. Building Ansible requires *easy_install*. easy_install is provided in the setuptools 18.0.1 Python package. This package can be installed like this: (**Only** for RHEL6, RHEL7 and SLES12)

         wget https://bootstrap.pypa.io/ez_setup.py
         sudo python ez_setup.py
       
5.  In order to install certain Python modules for Ansible later on, *pip* is required:

    On RHEL 6.6 and RHEL 7.1:
  
        sudo easy_install pip
      
	On SLES 11:
	
		sudo pip3 install --upgrade pip
		
    On SLES 12:

        sudo easy_install pip==1.2.1

6.  Clone the latest stable Ansible 2.1.0.0 GitHub source tree:

        cd /<source_root>/
        git clone https://github.com/ansible/ansible.git
        cd /<source_root>/ansible
        git checkout v2.1.0.0-1

7.  Setup the environment and get Ansible modules:

        source ./hacking/env-setup
        git submodule update --init --recursive
        source ./hacking/env-setup -q

8.  Ansible uses the following Python modules which need to be installed:

    On RHEL 6.6 and RHEL 7.1:
  
        sudo pip install paramiko PyYAML jinja2 httplib2 passlib nose mock mercurial six patch django call coverage==3.7.1 coveralls funcsigs pycrypto

    On SLES11:

        sudo pip install paramiko PyYAML jinja2 httplib2 passlib nose mock six patch call coverage==3.7.1 coveralls funcsigs pycrypto

    On SLES12:

		cd /<source_root>/
		wget https://pypi.python.org/packages/f7/83/377e3dd2e95f9020dbd0dfd3c47aaa7deebe3c68d3857a4e51917146ae8b/pyasn1-0.1.9.tar.gz#md5=f00a02a631d4016818659d1cc38d229a
		tar zxf pyasn1-0.1.9.tar.gz
		cd pyasn1-0.1.9/
		sudo python setup.py install
		cd /<source_root>/ansible         
		sudo pip install paramiko PyYAML jinja2 httplib2 passlib nose mock mercurial six patch django call coverage==3.7.1 coveralls funcsigs pycrypto
      
## Installation

1.  Run this command

        sudo make install

## Testing (Optional)
1. Install test pre-requisites

        cd /<source_root>/ansible
        sudo pip install -r ./lib/ansible/test-requirements.txt
    
2.  Now run the following command in ansible/ main directory

        make tests 

