<!---PACKAGE:Docker Compose--->
<!---DISTRO:RHEL 7.x:1.9--->
<!---DISTRO:SLES 12.x:1.9--->
<!---DISTRO:Ubuntu 16.x:Distro, 1.9--->

# Building Docker Compose  
Below versions of Docker Compose are available in respective distributions at the time of this recipe creation:   
*	RHEL 7.1/7.2/7.3 has 1.9.0  
*	SLES 12/12-SP1/12-SP2 has 1.9.0  
*	Ubuntu 16.04/16.10 has 1.9.0  

The instructions provided below specify the steps to build Docker Compose version  1.9.0 on IBM z Systems for 
RHEL 7.1/7.2/7.3, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.  

_**General Notes:**_   
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

##Building and Installing Docker Compose    
####1.1) Install dependencies  

*	RHEL 7.1/7.2/7.3  
```
sudo yum install -y python-setuptools
```
*	SLES 12
```
sudo zypper install -y python-setuptools python-xml
```
*	SLES 12-SP1 
```
sudo zypper install -y python-setuptools
```
*	SLES 12-SP2 
```
sudo zypper install -y python-pyOpenSSL python-setuptools
```
*	Ubuntu 16.04/16.10 
```
sudo apt-get update
sudo apt-get install -y python-pip
pip install --upgrade pip
```
	
####1.2) Install pip with easy_install (For RHEL 7.1/7.2/7.3 & SLES 12/12-SP1/12-SP2)
```
sudo easy_install pip
```
   
####1.3) Upgrade backports.ssl_match_hostname (For RHEL 7.1/7.2/7.3)
```
sudo pip install backports.ssl_match_hostname --upgrade
```
####1.4) Install docker-compose	
```
sudo pip install docker-compose==1.9.0
```
		
####1.5) To verify the Docker Compose installation
```
docker-compose version
```
        
##References:  
  https://github.com/docker/compose  
  https://docs.docker.com/compose/