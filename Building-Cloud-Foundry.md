<!---PACKAGE:Cloud Foundry--->
<!---DISTRO:UBUNTU 16.04:2--->

# Building Cloud Foundry V2

[Cloud Foundry V2](https://github.com/cloudfoundry/cf-release) is an open platform as a service (PaaS) that provides a choice of clouds, developer frameworks, and application services. Cloud Foundry makes it faster and easier to build, test, deploy, and scale applications. Cloud Foundry v237 has been built and tested on Linux on z Systems using [cf_nise_installer](https://github.com/yudai/cf_nise_installer).  This is a single node install best suited for Proof of concept purposes.  Production install using BOSH on OpenStack will be prioritized when Cloud Foundry moves to using Ubuntu 16.04 Xenial stemcells.  The following instructions can be used for Ubuntu 16.04.

### _**General Note:**_
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Building Prerequisites
#### Building warden root filesystems

##### Step 1 : Install the Dependencies
```
sudo apt-get update
sudo apt-get install aufs-tools git make build-essential docker.io
```
##### Step 2 : Checkout the code from repository
```
cd /<source_root>/
git clone https://github.com/cloudfoundry/stacks.git
cd /<source_root>/stacks
```

##### Step 3 : Modify following files

 1. Makefile


    *  `/<source_root>/stacks/Makefile`, add s390x support.
        ```diff
   	
	@@ -6,6 +6,9 @@	ifeq ("$(arch)","ppc64le")
        docker_image := "ppc64le/ubuntu:trusty"
        docker_file := cflinuxfs2/Dockerfile.$(arch)
        $(shell cp cflinuxfs2/Dockerfile $(docker_file))
        $(shell sed -i 's/FROM ubuntu:trusty/FROM ppc64le\/ubuntu:trusty/g' $(docker_file))
        + else ifeq ("$(arch)","s390x")
        +        docker_image := "s390x/ubuntu:16.04"
        +         docker_file := cflinuxfs2/Dockerfile.$(arch)
        else  
      ```

2 . install-packages.sh
 

    *  `/<source_root>/stacks/cflinuxfs2/build/install-packages.sh`, add s390x support.
         ```diff
             
@@ -11,6 +11,9 @@  function install_mysql_so_files() {
    mysqlpath="/usr/lib/x86_64-linux-gnu"
    if [ "`uname -m`" == "ppc64le" ]; then
        mysqlpath="/usr/lib/powerpc64le-linux-gnu"
    fi
+    if [ "`uname -m`" == "s390x" ]; then
+       mysqlpath="/usr/lib/s390x-linux-gnu"
+    fi
    apt_get install libmysqlclient-dev
    tmp=`mktemp -d`
    mv $mysqlpath/libmysqlclient* $tmp
    apt_get remove libmysqlclient-dev libmysqlclient18
    mv $tmp/* $mysqlpath/

@@ -57,7 +60,7 @@ libclass-accessor-perl
 libcups2
 libcurl3
 libcurl3-dev
-libcwidget3
+libcwidget3v5
 libdatrie1
 libdirectfb-1.2-9
 libdjvulibre-dev
@@ -66,7 +69,7 @@ libdjvulibre21
 libdrm-intel1
 libdrm-nouveau2
 libdrm-radeon1
-libept1.4.12
+libept1.5.0
 libfuse-dev
 libfuse2
 libgd2-noxpm-dev
@@ -77,17 +80,15 @@ libgtk-3-0
 libgtk-3-common
 libicu-dev
 libilmbase-dev
-libilmbase6
+libilmbase12
 libio-string-perl
 liblapack-dev
 libmagickcore-dev
 libmagickwand-dev
-libmariadbclient-dev
 libncurses5-dev
 libnl-3-200
 libopenblas-dev
 libopenexr-dev
-libopenexr6
 libpango1.0-0
 libparse-debianchangelog-perl
 libpcre3-dev
@@ -95,7 +96,7 @@ libpixman-1-0
 libpq-dev
 libreadline6-dev
 libsasl2-modules
-libsigc++-2.0-0c2a
+libsigc++-2.0-0v5
 libsqlite-dev
 libsqlite3-dev
 libssl-dev
@@ -103,8 +104,7 @@ libsub-name-perl
 libsysfs2
 libthai-data
 libthai0
-libts-0.0-0
-libxapian22
+libxapian22v5
 libxcb-render-util0
 libxcb-render0
 libxcomposite1
@@ -142,7 +142,6 @@ tasksel
 tasksel-data
 tcpdump
 traceroute
-tsconf
 ttf-dejavu-core
 unzip
 uuid-dev
@@ -152,6 +151,9 @@ zip
 if [ "`uname -m`" == "ppc64le" ]; then
 packages=$(sed '/\b\(libopenblas-dev\|libdrm-intel1\|dmidecode\)\b/d' <<< "${packages}")
 ubuntu_url="http://ports.ubuntu.com/ubuntu-ports"
+elif [ "`uname -m`" == "s390x" ]; then
+packages=$(sed '/\b\(libopenblas-dev\|libdrm-intel1\|dmidecode\)\b/d' <<< "${packages}")
+ubuntu_url="http://ports.ubuntu.com/ubuntu-ports"
 else
 ubuntu_url="http://archive.ubuntu.com/ubuntu"
 fi
 ```

##### Step 4 : Create a new file  ` /<source_root>/stacks/cflinuxfs2/Dockerfile.s390x ` with the below contents
```
FROM s390x/ubuntu:16.04

ADD build /tmp/build
ADD assets /tmp/assets

RUN mkdir -p /var/lib/resolvconf && touch /var/lib/resolvconf/linkified && \
bash /tmp/build/configure-locale-timezone.sh && \
bash /tmp/build/install-packages.sh && \
bash /tmp/build/generate-all-locales.sh && \
bash /tmp/build/install-ruby.sh 2.2.4 && \
bash /tmp/build/configure-core-dump-directory.sh && \
bash /tmp/build/configure-firstboot.sh && \
useradd -u 2000 -mU -s /bin/bash vcap && \
mkdir /home/vcap/app && \
chown vcap /home/vcap/app && \
ln -s /home/vcap/app /app && \
rm -rf /tmp/*

```

##### Step 5 : Create a rootfs for the cflinuxfs2 stack
```
make
```
This will create the ```cflinuxfs2.tar.gz``` file which will be required by Cloud Foundry in the later steps

#### Building Cloud Foundry Buildpacks
Please use recipe [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-CloudFoundry-Buildpacks) to build Cloud Foundry Buildpacks

## Building and Installing Cloud Foundry V2

### Install the Dependencies

    sudo apt update
	sudo apt install git curl openjdk-8-jdk golang libmysqlclient-dev libpq-dev
	
### Create and modify bootstrap.sh as follows
	mkdir /<source_root>/
	cd /<source_root>/
	git clone https://github.com/yudai/cf_nise_installer.git
    curl -s -k -B https://raw.githubusercontent.com/yudai/cf_nise_installer/${INSTALLER_BRANCH:-master}/scripts/bootstrap.sh > bootstrap.sh
	
#### Modify bootstrap.sh and comment out following 2 lines
    # exit 1
    # ./scripts/install.sh

Run **bootstrap.sh** to clone required cf_nise_installer files
```
chmod 755 bootstrap.sh
./bootstrap.sh
```

### Install Cloud Foundry V2 components and its dependencies
```
cd cf_nise_installer
sudo apt-get update
./scripts/install_ruby.sh
source ~/.profile
./scripts/clone_nise_bosh.sh
./scripts/clone_cf_release.sh
./scripts/install_environemnt.sh
```	
### Modify the **cf-237.yml release file** to use Golang 1.6 before starting main installation

```vi cf-release/releases/cf-237.yml```

Modify Golang 1.5 dependencies to Golang 1.6 - for example
	
	name: gorouter
		version: b7ee8aac577b18d69868cd89ffecbfe980bf7b07
		fingerprint: b7ee8aac577b18d69868cd89ffecbfe980bf7b07
		sha1: 7ea899613e3129c420ac4a4c994b1c344fea8b32
		dependencies:
		- golang1.6
	name: route_registrar
		version: 2ec8e115fd10d508f4c96123c92400dfef5ec801
		fingerprint: 2ec8e115fd10d508f4c96123c92400dfef5ec801
		sha1: 85b75bf5fec1381957206e9a1c0b5aacf4c410ef
		dependencies:
		- golang1.6

### Install Cloud Foundry V2 components in stages

1. Modify ```scripts/install_cf_release.sh``` and comment out
	- ```# sudo env PATH=$PATH bundle exec ./bin/nise-bosh --keep-monit-files -y ../cf-release ../manifests/deploy.yml micro_ng -n ${NISE_IP_ADDRESS}```
2. **Start the Cloud Foundry installation**
	- ```scripts/install_cf_release.sh```

Reinstall **Postgres** as follows
```
rm -rf /var/vcap/packages/postgres-9.4.6
cd /tmp
wget https://ftp.postgresql.org/pub/source/v9.4.6/postgresql-9.4.6.tar.gz
tar xvfz postgresql-9.4.6.tar.gz
cd postgresql-9.4.6
export BOSH_INSTALL_TARGET=/var/vcap/packages/postgres-9.4.6
./configure --prefix="${BOSH_INSTALL_TARGET}"
pushd src/bin/pg_config > /dev/null
make
make install
popd > /dev/null
cp -LR src/include "${BOSH_INSTALL_TARGET}"
pushd src/interfaces/libpq > /dev/null
make
make install
popd > /dev/null
pushd src > /dev/null
make
make install
popd > /dev/null
pushd contrib > /dev/null
make
make install
popd > /dev/null
cd <your work directory>/cf_nise_installer
rm -rf /var/vcap/data/packages/postgres-9.4.6/4ca292f2443cfc682e87632ebd15fb3f3fa53123/*
mv /var/vcap/packages/postgres-9.4.6/* /var/vcap/data/packages/postgres-9.4.6/4ca292f2443cfc682e87632ebd15fb3f3fa53123/
rm -rf /var/vcap/packages/postgres-9.4.6
ln -s /var/vcap/data/packages/postgres-9.4.6/4ca292f2443cfc682e87632ebd15fb3f3fa53123/ /var/vcap/packages/postgres-9.4.6
```
Ensure **Postgres** is installed by examining following folders
```
ls /var/vcap/packages/postgres-9.4.6
ls /var/vcap/packages/postgres-9.4.6/bin/
```		 
Continue install by making following changes in ```./scripts/install_cf_release.sh```
```
# ./scripts/generate_deploy_manifest.sh
# bundle install
# sudo env PATH=$PATH bundle exec ./bin/nise-bosh --keep-monit-files -y ../cf-release ../manifests/deploy.yml micro_ng -n ${NISE_IP_ADDRESS}
sudo env PATH=$PATH bundle exec ./bin/nise-bosh -y ../cf-release ../manifests/deploy.yml micro -n ${NISE_IP_ADDRESS}
```
**Resume installation**
```
./scripts/install_cf_release.sh
```
		
**Golang 1.6** will require following changes for s390x
	
  1. Install Golang 1.6 by running ```sudo apt-get install golang```
  2. Copy Go libraries to Cloud Foundry Go packages directory ```rsync -avH /usr/lib/go-1.6/*  /var/vcap/data/packages/golang1.6/85a489b7c0c2584aa9e0a6dd83666db31c6fc8e8/```
  3. Copy Go shared contents to Cloud Foundry Go packages directory ```cp -r /usr/share/go-1.6/* /var/vcap/data/packages/golang1.6/85a489b7c0c2584aa9e0a6dd83666db31c6fc8e8/```
  4. Create Golang 1.5 link to point to Golang 1.6: ln -s /var/vcap/packages/golang1.6 /var/vcap/packages/golang1.5

**Resume installation**
```
rm -rf /var/vcap/packages/gnatsd
./scripts/install_cf_release.sh
```

When **cloud_controller_ng** component fails please make following changes to ensure correct s390x mariadb packages are used

```
cp /usr/bin/mysql_config /var/vcap/packages/libmariadb/libmariadb/usr/bin
cp /usr/include/mysql/* /var/vcap/packages/libmariadb/libmariadb/usr/include/
cp -r /usr/include/mysql/* /var/vcap/packages/libmariadb/libmariadb/usr/include/mysql
cp -r /usr/lib/s390x-linux-gnu/ /var/vcap/packages/libmariadb/libmariadb/usr/lib/
cp /usr/share/aclocal/mysql.m4  /var/vcap/packages/libmariadb/libmariadb/usr/share/aclocal/
```

**Resume installation**
```
rm -rf /var/vcap/packages/cloud_controller_ng
./scripts/install_cf_release.sh
```
	
When all components are installed, additional changes are required in the following components

  1. **UAA**
	 ```
	 cp -r /usr/lib/jvm/java-8-openjdk-s390x/* /var/vcap/packages/uaa/jdk/
	 ```
  2. **DEA and Warden**
     - Modify and comment out "memsw" related sections of Ruby filesrb files in dea_next and warden package
	```
/var/vcap/data/packages/dea_next/f637d1bfd4c973fe31478c1fb35069c2c1cccc50/vendor/cache/warden-b72b8c5a532d/warden/spec/container/linux_spec.rb
/var/vcap/data/packages/dea_next/f637d1bfd4c973fe31478c1fb35069c2c1cccc50/vendor/cache/warden-b72b8c5a532d/warden/lib/warden/container/features/mem_limit.rb
/var/vcap/data/packages/warden/45feaaafb1ed6e77fd1ddd067b2577b8d13c03c7/warden/lib/warden/container/features/mem_limit.rb
	```
	 - Modify /var/vcap/data/packages/warden/45feaaafb1ed6e77fd1ddd067b2577b8d13c03c7/warden/root/linux/setup.sh as follows
	```
#mount -t cgroup -o $subsystem none $cgroup_path/$subsystem
mount -B /sys/fs/cgroup/$subsystem $cgroup_path/$subsystem
	```
  3. Replace **rootfs_cflinuxfs2** package contents with ```cflinuxfs2.tar.gz``` file created in the _Building Prerequisites_ step
    ```
cp /<rootfs build directory>/cflinuxfs2.tar.gz /var/vcap/packages/rootfs_cflinuxfs2/ (where cflinuxfs2.tar.gz is the rooftfs built earlier)
mkdir /var/vcap/packages/rootfs_cflinuxfs2/rootfs
tar xvfz /var/vcap/packages/rootfs_cflinuxfs2/cflinuxfs2.tar.gz -C /var/vcap/packages/rootfs_cflinuxfs2/rootfs
	```
	
  4. Modify **Warden** code as follows and recompile it
       ```
cd /var/vcap/data/packages/warden/45feaaafb1ed6e77fd1ddd067b2577b8d13c03c7/warden/src/wsh/
vi /var/vcap/data/packages/warden/45feaaafb1ed6e77fd1ddd067b2577b8d13c03c7/warden/src/wsh/wshd.c
//rv = mount_umount_pivoted_root("/mnt");
make clean
make
cp wshd /var/vcap/data/packages/warden/45feaaafb1ed6e77fd1ddd067b2577b8d13c03c7/warden/root/linux/skeleton/bin/
cp wsh /var/vcap/data/packages/warden/45feaaafb1ed6e77fd1ddd067b2577b8d13c03c7/warden/root/linux/skeleton/bin/
       ```

  5. Modify _hm9000_api_server_ctl_ code as follows
    ```
vi /var/vcap/jobs/hm9000/bin/hm9000_api_server_ctl
#hm9000.service.cf.internal
host consul.service.cf.internal
    ```
	   
  6. Rebuild [consul](https://github.com/linux-on-ibm-z/docs/wiki/Building-Consul) as per recipe and consul-template as follows
    - Building consul-template is similar to building consul except that you are checking out consul-template project
    - ```git clone https://github.com/hashicorp/consul-template.git```
	- Once generated, copy consul and consul-template binaries to ```/var/vcap/packages/consul/bin/```
	
  9. Reboot your system
	```
reboot
	```	
	
  7. Once system comes back online **Start Cloud Foundry** as follows
	```
mount --make-rprivate /
cd /<source_root>/cf_nise_installer
./scripts/start.sh
	```
	
  8. Upgrade default Buildpacks with Buildpacks generated in prerequisite step - for example
	```
cf update-buildpack nodejs_buildpack -p /<source_root>/buildpacks/nodejs_buildpack-cached-v1.5.18.zip
	```	

# Post install steps

  - Build and install cf tool
  ```
export GOPATH=/<source_root>/
go get github.com/cloudfoundry/cli
cd $GOPATH/src/github.com/cloudfoundry/cli
./bin/build  
  ```
  Copy 'cf' tool from ```$GOPATH/src/github.com/cloudfoundry/cli/out``` to a directory of your choosing and put it on the path
  
  - Once all components are running we can log in and update all buildpacks for s390x

```cf login -a https://api.<your ip>.xip.io -u admin -p c1oudc0w --skip-ssl-validation```

You are now ready to deploy applications to Cloud Foundry
	

# Note
It is important that all Cloud Foundry components start before you can deploy the applications. You can use ```sudo /var/vcap/bosh/bin/monit summary``` command to observe the status of all services. They should all be in *running* state.

When a component fails during install
	
  1. Delete the component folder from /var/vcap/packages (eg if go uaa fails then delete the /var/vcap/packages/uaa)
  2. Resume installation by running ```./scripts/install_cf_release.sh```

    
## Reference
https://github.com/cloudfoundry/cf-release

https://github.com/yudai/cf_nise_installer
