[Ansible](http://www.ansible.com/) 1.9.2 "Dancing In the Streets" has been ported to Linux on IBM z Systems. The following build instructions have been tested on RHEL 6, RHEL 7.1, SLES 11.3, and SLES 12.0.

## Building Ansible on RHEL 6, RHEL 7.1, SLES 11.3, and SLES 12.0

*Note: When following the steps below please use a standard permission user unless otherwise specified.*

1. Building Ansible requires *easy_install*. easy_install is provided in the setuptools 18.0.1 Python package. This package can be installed like this:

         wget https://bootstrap.pypa.io/ez_setup.py
         sudo python ez_setup.py
       
2.  In order to install certain Python modules for Ansible later on, *pip* is required:

    On RHEL 6, RHEL 7.1, and SLES 12.0:
  
        sudo easy_install pip
      
    On SLES 11.3:

        sudo easy_install pip==1.2.1

3.  Clone the latest stable Ansible 1.9 GitHub source tree.

    *Note: If you do not have Git installed, then install it like this:*
  
    *RHEL 6 and RHEL 7.1*
  
        sudo yum -y install git
      
    *SLES 11.3 and SLES 12.0*
  
        sudo zypper -y install git

    Now, clone Ansible like this:

        git clone https://github.com/ansible/ansible.git
        cd ansible
        git checkout stable-1.9

4.  Setup the environment and get Ansible modules:

        source ./hacking/env-setup
        cd ˜/src/github.com/ansible/ansible; git submodule update --init --recursive
        source ./hacking/env-setup -q

5.  Install *python-devel* like this:

    On RHEL 6 and RHEL 7.1:
  
        sudo yum install python-devel
      
    On SLES 11.3 and SLES 12.0:
  
        sudo zypper install python-devel
  
6.  Ansible uses the following Python modules which need to be installed:
  
        sudo pip install paramiko PyYAML Jinja2 httplib2 six
      
## Installation

1.  Run this command:

        sudo make install

## Testing (Optional)

1.  To run unit tests, first install the following required packages:

    On RHEL 6, RHEL 7.1, and SLES 12.0:
  
        sudo pip install paramiko PyYAML jinja2 httplib2 passlib nose mock mercurial coverage coveralls
      
    On SLES 11.3:
  
        sudo pip install paramiko PyYAML jinja2 httplib2 passlib nose mock mercurial coverage coveralls funcsigs

2.  Now run the following command in ansible/ main directory :

        make tests
  
    *Note: If the tests do not run then run the following command first:*
  
        sudo pip install --upgrade paramiko PyYAML jinja2 httplib2 passlib nose mock mercurial coverage coveralls
   
    *On SLES 11.3:*
  
        sudo pip install --upgrade paramiko PyYAML jinja2 httplib2 passlib nose mock mercurial coverage coveralls funcsigs

3.  Integration Tests (Full exhaustive test suite):

        cd test/
        cd integration/
        sudo -E make non_destructive
       