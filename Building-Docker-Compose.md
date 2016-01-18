# Building Docker Compose
Docker Compose 1.5.2 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1 and SLES 12

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##### 1. Install the following Dependencies
For RHEL7.1 :

      sudo yum update -y -qq
      sudo yum install -y git python-setuptools net-tools openssl-devel libffi-devel libxslt-devel libxml2-devel
        
For SLES12:

      sudo zypper install -y git libxslt-devel libxml2-devel python-setuptools libpipeline-devel python-lxml python-xml

##### 2. Install pip with easy_install
      sudo easy_install pip

##### 3. Create a working directory as a Docker Compose installation workspace  
      mkdir /<source_root>/
      cd /<source_root>/

##### 4. Get Docker Compose source from github
      git clone --branch 1.5.2 https://github.com/docker/compose.git
      cd /<source_root>/compose

##### 5. Build and Install Docker Compose
      sudo pip install -r requirements.txt
      sudo pip install -r requirements-dev.txt
        
      sudo python setup.py install

##### 6. Verify Docker Compose version
      docker-compose version
        
# References
      https://github.com/docker/compose
      https://docs.docker.com/compose/
        