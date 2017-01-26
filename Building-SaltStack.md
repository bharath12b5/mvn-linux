<!---PACKAGE:Salt--->
<!---DISTRO:RHEL 6.6:2016.11.1--->
<!---DISTRO:RHEL 7.1:2016.11.1--->
<!---DISTRO:SLES 11:2016.11.1--->
<!---DISTRO:SLES 12:2016.11.1--->
<!---DISTRO:Ubuntu 16.x:2016.11.1--->

# Building SaltStack

Below versions of SaltStack(Salt) are available in respective distributions at the time of this recipe creation:  

* Ubuntu 16.04 has  2015.8.8  
* Ubuntu 16.10 has  2016.3.1

The instructions provided below specify the steps to build SaltStack v2016.11.1 on IBM z Systems for RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General notes:**_ 

* _When following the steps below please use a standard permission user unless otherwise specified._  
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Step 1: Install the dependencies

* RHEL 6.8/7.1/7.2/7.3

	```
	sudo yum install git wget tar python-virtualenv.noarch make which svn bzip2 unzip gcc gcc-c++ python-devel make autoconf net-tools ssh python-setuptools python-lxml python-ldap sqlite-devel openldap-devel libxslt-devel openssl-devel libffi-devel openssl libtool libbz2-devel bzip2-devel.s390x
    sudo easy_install pip
	```

* SLES 11-SP4

	```
	sudo zypper install git tar bzip2 unzip python-devel make autoconf net-tools wget python-setuptools python-lxml python-ldap libxslt-devel gcc openssl libopenssl-devel libffi-devel python-xml gcc-c++ libbz2-devel awk libbz2-devel xz
    sudo easy_install pip
    sudo pip install 'cryptography==1.4'
	```

* SLES 12/12-SP1/12-SP2

	```
    sudo zypper install git tar bzip2 unzip python-devel make autoconf net-tools wget python-setuptools python-lxml python-ldap libxslt-devel gcc openssl libopenssl-devel libffi-devel python-xml gcc-c++ libbz2-devel
    sudo easy_install pip
	```

* Ubuntu 16.04/16.10

	```
	sudo apt-get install git wget tar make bzip2 unzip gcc python python-dev python-setuptools python-lxml python-ldap libxslt1-dev libffi-dev openssl libtool g++ libssl-dev 
    sudo easy_install pip
	```

* Install required packages using pip

    ```
    sudo pip install paramiko httplib2 passlib nose mock mercurial six patch django call coverage==3.7.1 coveralls funcsigs pyzmq PyYAML pycrypto msgpack-python jinja2 psutil futures tornado
    ```

### Step 2: Clone the repository and install SaltStack 

    cd /<source_root>/
    git clone https://github.com/saltstack/salt
    cd salt        
    git checkout v2016.11.1
    cd /<source_root>/
    sudo pip install -e ./salt  
    USE_SETUPTOOLS=1 sudo easy_install salt
  

### Step 3: Configure SaltStack to run self-contained version 

  
    mkdir -p /<source_root>/etc/salt/
    cd /<source_root>/
    cp ./salt/conf/master ./salt/conf/minion /<source_root>/etc/salt/
    cd /<source_root>/etc/salt/ 
  
   * Edit config file `master` as shown below 

    ```diff
    -	    #user: root
    +	    user: root
    -	    #root_dir: /
    +	    root_dir: /<src_root>/

    ```
    _**Note**:  Change the publish_port and ret_port values if required_  

   * Edit config file `minion` as shown below  

    ```diff
    -	    #master: salt
    +	    master: localhost
    -	    #user: root
    +	    user: root
    -	    #root_dir: /
    +	    root_dir: /<src_root>/

    ```
_**Note**: If the ret_port value in the master config file is changed then, set the same value to master_port value in the minion config file_

   * Start the master and minion, accept the minion's key, and verify your local Salt installation is working 

	```
    cd /<source_root>/
    sudo salt-master -c ./etc/salt -d 
    sudo salt-minion -c ./etc/salt -d 
    sudo salt-key -c ./etc/salt -L 
    sudo salt-key -c ./etc/salt -A 
    sudo salt -c ./etc/salt '*' test.ping  
	```

### Step 4: Unit and integration tests (optional)

* RHEL 6.8/SLES 11-SP4 (Install Python >= 2.7.x for better test results)

    * Python >= 2.7.x
      
	  Instructions for building Python can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Python-2.7.x)

    * Configure pip2.7 as pip:
      ```
      wget --no-check-certificate https://bootstrap.pypa.io/ez_setup.py  
      sudo /usr/local/bin/python2.7 ez_setup.py   
      sudo /usr/local/bin/easy_install-2.7 pip
      export PATH=/usr/local/bin:$PATH 
      ``` 

    * Install Python dependencies from step 1 (Install the dependencies) with newly installed Python 2.7 

    * Use pip install `--index-url=http://pypi.python.org/simple/ --trusted-host pypi.python.org <PackageName>` on SLES 11-SP4 to overcome SSL related issue.

* Test SaltStack

 	```
    cd /<source_root>/salt
    sudo pip install git+https://github.com/saltstack/salt-testing.git#egg=SaltTesting
    sudo pip install -r requirements/dev_python27.txt (Only for RHEL 6.8 and SLES 11-SP4 with Python 2.7.x)
    sudo pip install -r requirements/dev_python26.txt (Only for RHEL 6.8 and SLES 11-SP4 with Python 2.6.x)
    sudo ./setup.py test
	```

* Edit `/<source_root>/salt/tests/integration/__init__.py` file and run test again if minion test failed on RHEL 6.8 and SLES 11-SP4  

    ```diff
    @@ -618,7 +618,8 @@
                                 pass
                     del sock
                 elif isinstance(port, str):
    -                    joined = self.run_run('manage.joined', config_dir=self.config_dir)
    +                    #joined = self.run_run('manage.joined', config_dir=self.config_dir)
    +                    joined = self.run_run('manage.joined')
                     joined = [x.lstrip('- ') for x in joined]
                     if port in joined:
                         check_ports.remove(port)
    ```

_**Note:** Test failures seen in the following modules can be ignored as those are not related to IBM z Systems:              
            Module Tests, State Tests, Shell Tests, NetAPI Tests, Salt Unit Test_
  
### References: 

https://docs.saltstack.com/en/latest/topics/development/hacking.html   
https://docs.saltstack.com/en/latest/topics/installation/index.html