<!---PACKAGE:Ansible--->
<!---DISTRO:RHEL 6.6:2.0.2--->
<!---DISTRO:RHEL 7.1:2.0.2--->
<!---DISTRO:SLES 11:2.0.2--->
<!---DISTRO:SLES 12:2.0.2--->
[Ansible](http://www.ansible.com/) 2.0.2 has been ported to Linux on IBM z Systems. The following build instructions have been tested on RHEL 6.6, RHEL 7.1, SLES 11, and SLES 12.

## Building Ansible on RHEL 6.6, RHEL 7.1, SLES 11, and SLES 12

*Note: When following the steps below please use a standard permission user unless otherwise specified.*

1. Install dependencies:

    On RHEL 6.6 and RHEL 7.1:
  
        sudo yum install -y git wget tar svn bzip2 unzip gcc python-devel sudo make which autoconf net-tools ssh python-setuptools python-lxml python-ldap sqlite-devel openldap-devel libxslt-devel openssl-devel libffi-devel openssl
      
    On SLES 11:

         sudo zypper install -y git tar bzip2 unzip python-devel sudo make autoconf net-tools wget python-setuptools python-lxml python-ldap libxslt-devel gcc libffi-devel openssl

    On SLES 12:

         sudo zypper install -y git tar bzip2 unzip python-devel sudo make which autoconf net-tools wget python-setuptools python-lxml python-ldap libxslt-devel gcc openssl

             

2. Building Ansible requires *easy_install*. easy_install is provided in the setuptools 18.0.1 Python package. This package can be installed like this:

         wget https://bootstrap.pypa.io/ez_setup.py
         sudo python ez_setup.py
       
3.  In order to install certain Python modules for Ansible later on, *pip* is required:

    On RHEL 6.6 and RHEL 7.1:
  
        sudo easy_install pip
      
    On SLES 11 and SLES12:

        sudo easy_install pip==1.2.1

4.  Clone the latest stable Ansible 2.0.2 GitHub source tree:

        cd /<source_root>/
        git clone https://github.com/ansible/ansible.git
        cd /<source_root>/ansible
        git checkout v2.0.2.0-1

5.  Setup the environment and get Ansible modules:

        source ./hacking/env-setup
        git submodule update --init --recursive
        source ./hacking/env-setup -q

6.  Ansible uses the following Python modules which need to be installed:

    On RHEL 6.6 and RHEL 7.1:
  
        sudo pip install paramiko PyYAML jinja2 httplib2 passlib nose mock mercurial six patch django call coverage==3.7.1 coveralls funcsigs pycrypto

    On SLES11:

        sudo pip install paramiko PyYAML jinja2 httplib2 passlib nose mock mercurial six patch call coverage==3.7.1 coveralls funcsigs pycrypto

    On SLES12:

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

