<!---PACKAGE:Apache Mesos--->
<!---DISTRO:SLES 12:0.28.x--->
<!---DISTRO:RHEL 7.1:0.28.x--->
<!---DISTRO:Ubuntu 16.x:0.28.x--->

# Building Apache Mesos
Apache Mesos 0.28.x branch has been successfully built on Linux on z Systems. The following instructions can be used on RHEL 7, SLES 12(SP1) and Ubuntu 16.04.

##### General Notes:
i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory  ```/<source_root>/ ``` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it.

### 1. Install dependencies
SLES12(SP1):
```
sudo zypper install -y wget tar gcc gcc-c++ git patch java-1_8_0-openjdk-devel libzypp-devel libapr1 libapr1-devel subversion subversion-devel cyrus-sasl-devel cyrus-sasl-crammd5 python-devel libclang autoconf automake libtool
```

RHEL7:
```
sudo yum install -y make git tar wget java-1.8.0-openjdk-devel gcc gcc-c++ patch libzip-devel zlib-devel libcurl-devel apr apr-util apr-devel subversion subversion-devel cyrus-sasl-md5 python-devel which autoconf automake libtool
```
Ubuntu 16.04:
```
apt-get install -y tar wget git build-essential python-dev openjdk-8-jdk libcurl4-nss-dev libsasl2-dev libsasl2-modules maven libapr1-dev libsvn-dev zlib1g-dev libssl-dev autoconf automake libtool
```

##### Set environment variables on SLES12(SP1) and RHEL7 only:

SLES12(SP1):
```
export JAVA_HOME=/usr/lib64/jvm/java-1.8.0
export PATH=/usr/lib64/jvm/java-1.8.0/bin:$PATH
```
RHEL7:
```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0
export PATH=/usr/lib/jvm/java-1.8.0/bin:$PATH
```
##### Install Maven using the following steps:

```
cd /<source_root>/
wget http://apache.parentingamerica.com/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar zxvf apache-maven-3.3.9-bin.tar.gz
export PATH=/<source_root>/apache-maven-3.3.9/bin:$PATH
```

### 2. Configure, build and install Apache Mesos

Get the source

```
cd /<source_root>/
git clone https://github.com/linux-on-ibm-z/mesos
cd mesos/
git checkout 0.28.x
```

Configure and build Apache Mesos
```
cd /<source_root>/mesos
./bootstrap
mkdir build
cd build
../configure
make
```

Install Apache Mesos
```
cd /<source_root>/mesos/build
sudo make install
```

### 3. Run Apache Mesos
Start master
```
cd /<source_root>/mesos/build
./bin/mesos-master.sh --ip=<ip_address> --work_dir=/var/lib/mesos
```
Note: Ensure work directory exists and has required permissions

Start slave
```
cd /<source_root>/mesos/build
./bin/mesos-slave.sh --master=<ip_address>:5050
```

Web UI

Open "http://\<ip_address\>:5050" in your browser.

## Reference:

[http://mesos.apache.org/](http://mesos.apache.org/)

 