<!---PACKAGE:Ceilometer client--->
<!---DISTRO:SLES 12.x:2.5--->
<!---DISTRO:SLES 11.x:2.5--->
<!---DISTRO:RHEL 7.x:2.5--->
<!---DISTRO:RHEL 6.x:2.5--->
<!---DISTRO:Ubuntu 16.x:Distro, 2.5--->

# Building Python Ceilometer Client

Below versions of Python Ceilometer Client are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `2.4.0`

The instructions provided below specify the steps to build Ceilometer Client 2.5.0 on Linux on the IBM z Systems for RHEL 7.1/7.2/6.7, SLES 11-SP3, SLES 12/12-SP1 and Ubuntu 16.04.

**General Notes:**

i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `<source_root>` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

1. Install the build dependencies

 For RHEL 6.7 
 
        sudo yum install -y git python-devel python-setuptools python-setuptools-devel python-virtualenv findutils gcc openssl-devel xz curl wget tar libffi-devel python-devel openssl-devel libffi python-cffi
 
 For RHEL 7.1/7.2

		sudo yum install -y git python-devel python-setuptools python-setuptools-devel python-virtualenv libffi-devel python-devel openssl-devel libffi python-cffi findutils gcc  libffi
		          
 For SLES 11-SP3
 
        sudo zypper install -y git-core wget gcc python-devel zlib-devel xz openssl-devel curl tar findutils ncurses-devel libbz2-devel libffi-devel  
 
 For SLES 12/12-SP1

		sudo zypper install -y git-core wget gcc zlib-devel findutils python-devel python-cffi openssl-devel libffi-devel tar

 For Ubuntu 16.04
                
        sudo apt-get update
        sudo apt-get install git python-dev findutils gcc python-setuptools python-dev build-essential libssl-dev libffi-dev


2. Install Python 2.7.9 as a dependency (For RHEL 6.7 & SLES 11-SP3 Only) 

   Instructions for building Python 2.7.9 can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Python-2.7.9)
    
		
3. Install Python modules dependencies

	3.1. Install pip 
	
	   RHEL 6.7
		
			curl -o /tmp/ez_setup.py https://bootstrap.pypa.io/ez_setup.py
			sudo /usr/local/bin/python2.7 /tmp/ez_setup.py   
			sudo /usr/local/bin/easy_install pip  
			sudo rm -f /usr/bin/pip
			sudo ln -s /usr/local/bin/pip /usr/bin/pip		

        
	   RHEL 7.1/7.2/Ubuntu 16.04
		
			sudo easy_install pip

	   SLES 11-SP3
		
			curl -o /tmp/ez_setup.py https://bootstrap.pypa.io/ez_setup.py
			sudo /usr/local/bin/python2.7 /tmp/ez_setup.py
			sudo /usr/local/bin/easy_install pip==1.2.1
			sudo rm -f /usr/bin/pip
			sudo ln -s /usr/local/bin/pip /usr/bin/pip	
		
	   SLES 12/12-SP1
       
			cd /<source_root>/
			rm -f ez_setup.py*
			wget https://bootstrap.pypa.io/ez_setup.py && sudo python ez_setup.py	   
			sudo easy_install pip==1.2.1

		

	3.2. Use pip to install other dependencies 
	  
	   RHEL 6.7/RHEL 7.1/7.2/Ubuntu 16.04   
		
			sudo pip install pbr virtualenv cryptography 
			
	   SLES 12/12-SP1   
		
			sudo pip install funcsigs docutils jinja2 extras pyrsistent unittest2 testtools cryptography 
			
	   SLES 11-SP3    
		
			sudo pip install funcsigs docutils jinja2 pytz extras pyrsistent unittest2 testtools imagesize babel coverage  mock oslosphinx reno python-subunit sphinx testrepository
		  
4. Install Netifaces(only for SLES 12/12-SP1 and SLES 11-SP3)

		cd /<source_root>/
        wget -O "netifaces-0.10.4.tar.gz" "https://pypi.python.org/packages/source/n/netifaces/netifaces-0.10.4.tar.gz#md5=36da76e2cfadd24cc7510c2c0012eb1e"
		tar xvzf netifaces-0.10.4.tar.gz
        cd netifaces-0.10.4 && sudo python setup.py install

5. Download the version 2.5.0 of Python-Ceilometerclient

		cd /<source_root>/
        git clone https://github.com/openstack/python-ceilometerclient.git
		cd python-ceilometerclient
        git checkout 2.5.0

6. Install Python-Ceilometerclient's Python requirements
        
		sudo pip install -r requirements.txt         
      
7. (Optional) Install test requirements and test

	SLES 11-SP3 Only 
		
		sudo pip install linecache2
		sudo pip install unittest2
     
	**NOTE:** pip 1.2.1 install on SLES 11-SP3 sometimes has issues - if one of these fails to install, simply run the install a second time and it will complete
		
	
	For all Platforms (except SLES 11-SP3)
		
		
		sudo pip install -r test-requirements.txt

	For all Platforms 
		
        python setup.py test
			

8. Install and verify
        
    RHEL7.1/7.2 / SLES12/12-SP1/ Ubuntu 16.04
	
		sudo python setup.py install
		
    RHEL6.7 / SLES11-SP3 
	
        sudo /usr/local/bin/python2.7 setup.py install
	
    Verify using the below command	
	
		ceilometer --help
      
		   
### References:
https://github.com/openstack/python-ceilometerclient/		   


