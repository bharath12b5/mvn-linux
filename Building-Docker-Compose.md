# Recipe for building Docker Compose
    The following build instructions describe how to build docker-compose on RHEL7 and SLES12 for IBM z Systems.

## Version
        1.4.0rc2

## Section 1: Install the following Dependencies
#### RHEL7:

        yum update -y -qq
        yum install -y \ 
            git \ 
            python-setuptools \ 
            net-tools \ 
            openssl-devel \ 
            libffi-devel \ 
            libxslt-devel \ 
            libxml2-devel  
        
        easy_install pip

#### SLES12:

        zypper install -y  \
                git \
                libxslt-devel \
                libxml2-devel \
                python-setuptools \ 
                libpipeline-devel \ 
                python-lxml \
                python-xml
                
        easy_install pip

## Section 2: Get docker-compose source from github
        cd /
        git clone --branch 1.4.0rc2 https://github.com/docker/compose.git
        cd compose

## Section 3: Build and Install
        pip install -r requirements.txt
        pip install -r requirements-dev.txt
        
        python setup.py install

## Section 4: Verify docker-compose version
        docker-compose version
        
# References
        https://github.com/docker/compose
        https://docs.docker.com/compose/
        