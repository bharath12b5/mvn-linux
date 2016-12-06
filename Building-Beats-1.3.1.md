# Building Beats 1.3.1

The instructions provided below specify the steps to build Beats 1.3.1 on IBM z Systems for RHEL 7.2 , SLES 12-SP1 and Ubuntu 16.04 by following these instructions.

_**General Notes:**_  

* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._


### Prerequisites 
 
* Go 1.7.1 (RHEL 7.2 , SLES 12-SP1 and Ubuntu 16.04)

       -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7).

* Docker >=1.10.0 (RHEL 7.2 , SLES 12-SP1) 

       -- Instructions for building Docker can be found [here](http://www.ibm.com/developerworks/linux/linux390/docker.html).

* Python >=2.7.9 (RHEL 7.2 , SLES 12-SP1) 

      -- Instructions for building Python can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Python-2.7.x).

* Docker-Compose >= 1.7.0  (RHEL 7.2 , SLES 12-SP1 and Ubuntu 16.04)

      -- Instructions for building Docker-Compose can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Docker-Compose).
 

### Step 1: Install the Dependencies 

* RHEL 7.2 

    ```
    sudo yum install -y git make wget tar gcc libpcap.s390x libpcap-devel.s390x  python-setuptools
    ```
   * Install virtualenv for Python
   ``` 
   sudo easy_install pip
   sudo pip install virtualenv 
   ````
   
* SLES 12-SP1

    ```
    sudo zypper install -y git make wget tar gcc libpcap.s390x libpcap-devel.s390x  python-setuptools
    ```
   * Install virtualenv for Python
   ``` 
   sudo easy_install pip
   sudo pip install virtualenv 
   ````    

* Ubuntu 16.04 

    ```
    sudo apt-get update
    sudo apt-get install -y git make wget tar gcc docker python python-setuptools libcap-dev libpcap0.8-dev
    ```
   * Install virtualenv for Python
   ``` 
   sudo easy_install pip
   sudo pip install virtualenv 
   ```` 
   
### Step 2: Download Beats 
   ```
    export GOPATH=$HOME
    mkdir -p $GOPATH/src/github.com/elastic
    cd $GOPATH/src/github.com/elastic
    git clone https://github.com/elastic/beats.git
    cd beats
    git checkout v1.3.1
   ```
   
### Step 3: Compile, test and generate configuration files for Beats
Currently there are 4 Beats: filebeat, packetbeat, topbeat and libbeat.
To compile, test and generate configuration files for a particular Beat use the commands as shown below. 
  ```
  cd $GOPATH/src/github.com/elastic/beats/<beat_name>
  make
  make unit-tests
  make system-tests
  make integration-tests
  make update 
  ```
  _**Note:**_ libbeat is used as a base for developing customized Beats. Hence, ```make update``` cannot be executed for this beat.

For example, for Packetbeat:

 ```
  cd $GOPATH/src/github.com/elastic/beats/packetbeat
  make
  make unit-tests
  make system-tests
  make integration-tests
  make update 
  ```

### Step 4: To start individual Beats
To run a Beat use the command as shown below. 
  ```
  cd $GOPATH/src/github.com/elastic/beats/<beat_name>
  sudo ./<beat_name> -e -c <path_to_beatname.yml_file> -d "publish" 
  ```
For example, for Packetbeat:
  ```
  cd $GOPATH/src/github.com/elastic/beats/packetbeat
  sudo ./packetbeat -e -c  <path_to_packetbeat.yml> -d "publish"
  ```
  _**Note:**_ Above command cannot be used for libbeat.
  
###Reference:
https://github.com/elastic/beats/blob/v1.3.1/CONTRIBUTING.md