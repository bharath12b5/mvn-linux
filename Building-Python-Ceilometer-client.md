# Building Python Ceilometer Client

Ceilometer Client 1.5.1 has been successfully built and tested on Linux on z Systems. The following instructions can be used for RHEL 7.1/6.6 and SLES 12/11.

_**General Notes:**_

i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `<source_root>` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

# Building Python-Ceilometerclient

1. Install the build dependencies

 For RHEL 6.6 
 
        sudo yum install -y git python-devel python-setuptools python-setuptools-devel python-virtualenv findutils gcc openssl-devel xz curl wget tar
 
 For RHEL 7.1

		sudo yum install -y git python-devel python-setuptools python-setuptools-devel python-virtualenv findutils gcc  
		          
 For SLES 11 
 
        sudo zypper install -y git-core wget gcc python-devel zlib-devel xz openssl-devel curl tar findutils ncurses-devel libbz2-devel
 
 For SLES 12

		sudo zypper install -y git-core wget gcc python-devel zlib-devel findutils


2. Install Python 2.7.9 as a dependency (For RHEL 6.6 & SLES 11 Only) 

   Instructions for building Python 2.7.9 can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Python-2.7.9)
    
		
3. Install Python modules dependencies

   3.1. Install pip 
	
      RHEL 6.6: 
		
		curl -o /tmp/ez_setup.py https://bootstrap.pypa.io/ez_setup.py
		sudo /usr/local/bin/python2.7 /tmp/ez_setup.py   
		sudo /usr/local/bin/easy_install pip  
		sudo rm -f /usr/bin/pip
        sudo ln -s /usr/local/bin/pip /usr/bin/pip		

        
      RHEL 7.1:
		
		sudo easy_install pip

      SLES 11: 
		
		curl -o /tmp/ez_setup.py https://bootstrap.pypa.io/ez_setup.py
		sudo /usr/local/bin/python2.7 /tmp/ez_setup.py
		sudo /usr/local/bin/easy_install pip==1.2.1
		sudo rm -f /usr/bin/pip
		sudo ln -s /usr/local/bin/pip /usr/bin/pip	
		
      SLES 12:
       
        cd /<source_root>/
		rm -f ez_setup.py*
		wget https://bootstrap.pypa.io/ez_setup.py && sudo python ez_setup.py	   
		sudo easy_install pip==1.2.1

		

   3.2. Use pip to install other dependencies 
	  
	  RHEL 6.6/RHEL 7.1:
		
		sudo pip install pbr
		sudo pip install virtualenv
	    
	  
	  SLES 11/SLES 12:
		
		sudo pip install funcsigs
        sudo pip install docutils
		sudo pip install jinja2

4. Download the version 1.5.1 of Python-Ceilometerclient

		cd /<source_root>/
        git clone https://github.com/openstack/python-ceilometerclient.git
		cd python-ceilometerclient
        git checkout 1.5.1
		  

		  
5. Install Python-Ceilometerclient's Python requirements
        
		sudo pip install -r requirements.txt         


6. (Optional) Install test requirements and test

	SLES 11 Only 
		
		sudo pip install linecache2
		sudo pip install unittest2
     
	**NOTE:** pip 1.2.1 install on SLES 11 sometimes has issues - if one of these fails to install, simply run the install a second time and it will complete
		
	
	For all Platforms 
		
		
		sudo pip install -r test-requirements.txt
		./run_tests.sh -N
	
	**NOTE:** SLES 12 reports a syntax error while installing unittest2. However, the Client successfully installs and passes all 252 tests


7. Install and verify
        
    RHEL7 / SLES12 
	
		sudo python setup.py install
		
    RHEL6 / SLES11 
	
        sudo /usr/local/bin/python2.7 setup.py install
	
    Verify using the below command	
	
		ceilometer --help
      
		   
### References:
https://github.com/openstack/python-ceilometerclient/		   


