<!---PACKAGE:Docker Compose--->
<!---DISTRO:RHEL 7.1:1.8--->
<!---DISTRO:SLES 12:1.8--->
<!---DISTRO:Ubuntu 16.x:Distro, 1.8--->

# Installing Docker Compose

Below versions of Docker Compose are available in respective distributions at the time of this recipe creation:

*    RHEL 7.1/7.2 has `1.8.1`
*    SLES 12/12-sp1 has `1.8.1`
*    Ubuntu 16.04 has `1.8.1`

The instructions provided below specify the steps to install Docker Compose version 1.8.1 on Linux on the IBM z Systems for RHEL 7.1/7.2, SLES 12/12-SP1 and Ubuntu 16.04.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### 1. Install the following Dependencies
For RHEL7.1/7.2 
```
sudo yum install -y python-setuptools
```
For SLES12
```
sudo zypper install -y python-setuptools python-xml
```
For SLES12-SP1
```
sudo zypper install -y python-setuptools
```
For Ubuntu16.04
```
sudo apt-get update
sudo apt-get install -y python-pip
pip install --upgrade pip
```
	
### 2. Install pip with easy_install (For RHEL7.1/7.2 & SLES 12/12-SP1)
```
sudo easy_install pip
```
   
### 3. Upgrade backports.ssl_match_hostname(For RHEL7.1/7.2)
```
sudo pip install backports.ssl_match_hostname --upgrade
```
### 4. Install docker-compose	
```
sudo pip install docker-compose
```
		
### 5. To verify the Docker Compose installation
```
docker-compose version
```
        
# References  
  https://github.com/docker/compose  
  https://docs.docker.com/compose/