# Building Beats

Beats 5.0.0 can be built and tested on Linux on z Systems for RHEL 7.2 by following these instructions.

_**General Notes:**_  

i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._



### Prerequisites (RHEL 7.2)
 
* Go 1.7.1
       -- Instructions for building Go can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7).

* Docker >=1.10.0
       -- Instructions for building Docker can be found [here](http://www.ibm.com/developerworks/linux/linux390/docker.html).

* Python >=2.7.9
      -- Instructions for building Python can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Python-2.7.x).

* Docker-Compose >= 1.7.0
      -- Instructions for building Docker-Compose can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Docker-Compose).
 

 
### Step 1 : Install the Dependencies 
* RHEL 7.2

    ```
     sudo yum install -y git make wget tar gcc libpcap.s390x libpcap-devel.s390x  python-setuptools
    ```
   * Install virtualenv for Python
   ``` 
       sudo easy_install pip
       sudo pip install virtualenv 
   ````    
    

### Step 2 : Build and Install Beats 
```
export GOPATH=$HOME
mkdir -p $GOPATH/src/github.com/elastic
cd $GOPATH/src/github.com/elastic
git clone https://github.com/elastic/beats.git
cd beats
git checkout v5.0.0
```
Currently there are 5 Beats: filebeat, packetbeat, metricbeat, winlogbeat, libbeat.
Compile and generate configuration files for a particular Beat by using the commands as shown below. 

* For Packetbeat:
```
cd $GOPATH/src/github.com/elastic/beats/packetbeat
make
make update
```

* For Winlogbeat:
```
cd $GOPATH/src/github.com/elastic/beats/winlogbeat
make
make update
```

* For Filebeat:
```
cd $GOPATH/src/github.com/elastic/beats/filebeat
make
make update
```

* For Metricbeat:
```
cd $GOPATH/src/github.com/elastic/beats/metricbeat
make
make update
```

* For Libbeat:
```
cd $GOPATH/src/github.com/elastic/beats/libbeat
make
make update
```


###Reference:
https://github.com/elastic/beats/blob/v5.0.0/CONTRIBUTING.md
