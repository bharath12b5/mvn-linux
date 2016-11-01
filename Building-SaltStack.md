# Building SaltStack [v2016.3.3]

The following build instructions have been tested with SaltStack v2016.3.3 on RHEL 7.2 on IBM z Systems.

_**General notes:**_ 

i) When following the steps below please use a standard permission user unless otherwise specified.  
ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it.

### Step 1 : Install the Dependencies

* RHEL 7.2
        
		sudo yum install git wget tar python-virtualenv.noarch make which svn bzip2 unzip gcc gcc-c++ python-devel make autoconf net-tools ssh python-setuptools python-lxml python-ldap sqlite-devel openldap-devel libxslt-devel openssl-devel libffi-devel openssl libtool 

     1. #### Install ```pip``` using below command
           `sudo easy_install pip`

     2. #### Few packages that are missing from yum repo can be install using pip
             sudo pip install paramiko httplib2 passlib nose mock mercurial six patch django call coverage==3.7.1 coveralls funcsigs pyzmq PyYAML pycrypto msgpack-python jinja2 psutil futures tornado
           
### Step 2 : Clone the repository and Install SaltStack

       cd /<source_root>/
       git clone https://github.com/saltstack/salt
       cd salt        
       git checkout v2016.3.3
       cd ..
       sudo pip install -e ./salt
               or 
       USE_SETUPTOOLS=1 sudo easy_install salt

### Step 3 : Configure SaltStack to run self-contained version

       mkdir -p /<source_root>/etc/salt/
       cd /<source_root>/
       cp ./salt/conf/master ./salt/conf/minion /<source_root>/etc/salt/
       cd /<source_root>/etc/salt/ 

   1. Edit master Conf File  
      * Uncomment and change the `user: root` value to your own user (e.g. user: test)
      * Uncomment and change the `root_dir: /` value to point to `/<source_root>/`
      * Change the publish_port and ret_port values if required

   2. Edit minion Conf File  
      * Uncomment and change the `user: root` value to your own user (e.g. user: test)
      * Uncomment and change the `root_dir: /` value to point to `/<source_root>/`
      * Uncomment and change the master: salt value to point at localhost
      * If the ret_port value in the master config file is changed than, set the same value to master_port value in the minion config file

   3. Start the master and minion, accept the minion's key, and verify your local Salt installation is working**

        cd /<source_root>/
        salt-master -c ./etc/salt -d 
        salt-minion -c ./etc/salt -d 
        salt-key -c ./etc/salt -L 
        salt-key -c ./etc/salt -A 
        salt -c ./etc/salt '*' test.ping  
      
### Step 4 : Unit and Integration tests (optional)
    
       cd /<source_root>/salt
       sudo pip install git+https://github.com/saltstack/salt-testing.git#egg=SaltTesting	
       sudo ./setup.py test

_**Note:** Ignore the test cases failures as these are not related to Linux on z system._ 
  
### References: 

https://docs.saltstack.com/en/latest/topics/development/hacking.html   
https://docs.saltstack.com/en/latest/topics/installation/index.html  
http://gitpython.readthedocs.io/en/stable/intro.html  
http://zeromq.org/intro:get-the-software 