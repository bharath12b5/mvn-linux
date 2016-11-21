<!---PACKAGE:Ansible--->
<!---DISTRO:RHEL 6.6:2.2.x--->
<!---DISTRO:RHEL 7.1:2.2.x--->
<!---DISTRO:SLES 11:2.2.x--->
<!---DISTRO:SLES 12:2.2.x--->
<!---DISTRO:Ubuntu 16.x:2.2.x--->

## Building Ansible

Below versions of Ansible are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `	2.2.0.0-1`
*    Ubuntu 16.10 has ` 2.2.0.0-1`

Latest Ubuntu builds are available [in a PPA here](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-apt-ubuntu).

*    _**Note:**_  Following the step `sudo apt-add-repository ppa:ansible/ansible`, make below changes in file`/etc/apt/sources.list.d/ansible-ubuntu-ansible-yakkety.list`.  
    Replace  

    ```
    deb http://ppa.launchpad.net/ansible/ansible/ubuntu yakkety main
    ```     			 

    with           			

    ```
    deb http://ppa.launchpad.net/ansible/ansible/ubuntu xenial main
    ```    

    Above change is required for Ansible ppa repository to use xenial as there is no release avaialble for yakkety yet.


The instructions provided below specify the steps to build [Ansible](http://www.ansible.com/) version 2.2.0.0 on Linux on the IBM z Systems for RHEL 7.1/7.2/7.3 and SLES 12/12-SP1.


_**General Notes:**_ 	 
* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

1. Install dependencies:

    On RHEL 7.1/7.2/7.3:
  
        sudo yum install -y git wget tar svn bzip2 unzip gcc python-devel make which autoconf net-tools ssh python-setuptools python-lxml python-ldap sqlite-devel openldap-devel libxslt-devel openssl-devel libffi-devel openssl

    On SLES 12/12-SP1:

        sudo zypper install -y git tar bzip2 unzip python-devel make which autoconf net-tools wget python-setuptools python-lxml python-ldap libxslt-devel gcc openssl libopenssl-devel libffi-devel python-xml

		
2.  Install Docker 1.10.0
	See instructions [here](http://www.ibm.com/developerworks/linux/linux390/docker.html)
	
	On SLES 12-SP1
		
		sudo zypper install -y docker
		
3.  In order to install certain Python modules for Ansible later on, *pip* is required:

   
    On RHEL 7.1/7.2/7.3:
	
		wget https://bootstrap.pypa.io/get-pip.py
		sudo python get-pip.py 
	        

    On SLES 12/12-SP1:

        wget https://pypi.python.org/packages/c3/a2/a63244da32afd9ce9a8ca1bd86e71610039adea8b8314046ebe5047527a6/pip-1.2.1.tar.gz
		tar zxvf pip-1.2.1.tar.gz
		cd pip-1.2.1
		sudo python setup.py install
		
		
4.  Clone the latest stable Ansible 2.2.0.0 GitHub source tree:

        cd /<source_root>/
        git clone https://github.com/ansible/ansible.git
        cd /<source_root>/ansible
        git checkout v2.2.0.0-1

5.  Setup the environment and get Ansible modules:

        source ./hacking/env-setup
        git submodule update --init --recursive
        source ./hacking/env-setup -q

6.  Ansible uses the following Python modules which need to be installed:

    On RHEL 7.1/7.2/7.3:
  
        sudo pip install paramiko PyYAML jinja2 httplib2 passlib nose mock mercurial six patch django call coverage==3.7.1 coveralls funcsigs pycrypto boto3 botocore

    On SLES 12/12-SP1:

		cd /<source_root>/
		wget https://pypi.python.org/packages/f7/83/377e3dd2e95f9020dbd0dfd3c47aaa7deebe3c68d3857a4e51917146ae8b/pyasn1-0.1.9.tar.gz#md5=f00a02a631d4016818659d1cc38d229a
		tar zxf pyasn1-0.1.9.tar.gz
		cd pyasn1-0.1.9/
		sudo python setup.py install
		cd /<source_root>/ansible 
        sudo pip install --upgrade setuptools        
		sudo pip install paramiko PyYAML jinja2 httplib2 passlib nose mock mercurial six patch django call coverage==3.7.1 coveralls funcsigs pycrypto boto3 botocore
    
## Installation

1.  Run this command

        sudo make install

## Testing (Optional)
1. Install test pre-requisites

        cd /<source_root>/ansible
        sudo pip install -r ./lib/ansible/test-requirements.txt
    
2.  Now run the following command in ansible/ main directory

        make tests 

