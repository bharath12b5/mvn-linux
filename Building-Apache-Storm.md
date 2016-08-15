<!---PACKAGE:Apache Storm--->
<!---DISTRO:SLES 12:1.0.1--->
<!---DISTRO:RHEL 7.1:1.0.1--->

# Apache Storm
Apache Storm is a distributed computation framework used for realtime computations.  Apache Storm 1.0.1 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 7.1, SLES 12.
  
### Prerequisites:  
Maven v3.0.0 or above -- Instructions for building Maven can be  found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven).  
  
### Obtaining Build Dependencies  
  
Step 1: Install the Dependencies

For RHEL 7.1  
```
sudo yum install git java-1.8.0-openjdk-devel.s390x tar  
```
For SLES 12  
```
sudo zypper install git java-1.8.0-openjdk tar  
```

Step 2: Install nodejs sdk from  https://developer.ibm.com/node/sdk/ for [**Linux on System z 64-bit**](http://www14.software.ibm.com/cgi-bin/weblap/lap.pl?popup=Y&la_formnum=&li_formnum=L-PIVN-A98LMB&title=IBM%20SDK%20for%20Node.js%20Version%206&accepted_url=http://public.dhe.ibm.com/ibmdl/export/pub/systems/cloud/runtimes/nodejs/6.2.2.0/linux/s390x/ibm-6.2.2.0-node-v6.2.2-linux-s390x.bin).  
  
Step 3: Install rvm and nvm
```
curl -L https://get.rvm.io | bash -s stable --autolibs=enabled && source ˜/.profile  
command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -  
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.26.1/install.sh | bash && source ˜/.bashrc
```
  
### Building Apache Storm   
Step 1: Set environment variables and working directory  

Add the nodejs executable to the path.  
```
export PATH=$PATH:<Nodejs Install Directory>/node/bin  
```

Setup Working Directory  
```
cd <source_root>
export WORK_DIR=`pwd`  
```

Setup up Maven and JAVA  
```
export JAVA_HOME=/usr/lib64/jvm/java  
export M2_HOME=<Your Maven location>
export PATH=$JAVA_HOME/bin:$PATH:$M2_HOME/bin
export _JAVA_OPTIONS=-Xmx4096m
export JVM_ARGS="-Xms4096m -Xmx4096m"
```  
  
Step 2: Clone and build  
```
cd <source_root>
git clone https://github.com/apache/storm.git  
cd storm  
git checkout tags/v1.0.1  
mvn clean install  
```

### Binaries  
To Create the binary distribution use:  
```
cd storm-dist/binary && mvn package  
```

The binaries can be found at :  
/&lt;source_root&gt;/storm-dist/binary/target/
