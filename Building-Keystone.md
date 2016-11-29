# Building Keystone

Below versions of Keystone is available in respective distributions at the time of this recipe creation  

* Ubuntu 16.04 has `9.0.0`

The instructions provided below specify the steps to build Keystone stable/Mitaka (9.2.0) on IBM z Systems for RHEL 7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04

_**General notes:**_ 

* When following the steps below please use a standard permission user unless otherwise specified 
* A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

### Step 1: Install the dependencies

* RHEL 7.2/7.3  

	```
	sudo yum install gcc git python-setuptools python-lxml curl python-ldap sqlite-devel openldap-devel python-devel libxslt-devel openssl-devel MySQL-python net-tools libffi-devel which openssl httpd memcached python-memcached mod_wsgi mariadb mariadb-server    
    sudo easy_install pip
	```
* SLES 12-SP1/12-SP2  

	```
	sudo zypper install gcc git-core python-setuptools python-lxml curl python-ldap openldap2-devel libffi-devel openssl-devel python-devel libxslt-devel python-mysql net-tools which openssl apache2 apache2-devel python-xml mariadb    
    sudo easy_install pip    
    sudo pip install mod_wsgi python-memcached functools32
	```
* Ubuntu 16.04  

	```
	sudo apt-get update 
	sudo apt-get install -y build-essential libncurses-dev git wget cmake gcc make tar libpcre3-dev bison scons libboost-dev libboost-program-options-dev openssl dh-autoreconf libssl-dev python-setuptools python-lxml curl python-ldap python-dev libxslt-dev net-tools libffi-dev apache2-dev python-mysqldb apache2 libapache2-mod-wsgi mysql-server    
    sudo easy_install pip    
    sudo pip install mod_wsgi python-memcached functools32
	```

### Step 2: Configure and start MariaDB server

Edit configuration File `/etc/my.cnf`. Update [mysqld] section, as below (Only for RHEL 7.2/7.3): 

```diff
@@ -1,6 +1,11 @@
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
+default-storage-engine = innodb
+innodb_file_per_table
+collation-server = utf8_general_ci
+init-connect = 'SET NAMES utf8'
+character-set-server = utf8
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
```

Initialize MariaDB server

* RHEL 7.2/7.3, SLES 12-SP1/12-SP2 
    ```
    sudo /usr/bin/mysql_install_db --user=mysql
    ```

* Ubuntu 16.04
    ```
    sudo /usr/sbin/mysqld --initialize --user=mysql --datadir=/var/lib/mysql/data
    ```

Start MariaDB service  

    sudo mkdir -p  /var/log/mysql 	(SLES 12-SP1/12-SP2 only)
    sudo /usr/bin/mysqld_safe --user=mysql & 

	
## Step 3: Create user and grant privileges on Keystone database

_**Note**_:   
*   ```<KEYSTONE_HOST_IP>```- IP of your machine where you are installing Keystone Service
*   ```<DB_HOST> ``` - IP or HostName of machine,where the MariaDB service is running
*   ```<KEYSTONE_DBPASS>``` -  database password for Keystone
*   ```<PASSWORD>``` - database password for root user

Follow below instruction to create Keystone database and grant required privileges

* Connect to MySQL using your credentials  

	```
	mysql -u root -h <DB_HOST> -p 
	```
*  Create database, grant privileges to "keystone" user and exit  

	```
	CREATE DATABASE keystone;
	GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY '<KEYSTONE_DBPASS>';
	GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY '<KEYSTONE_DBPASS>';
	exit
	```  
	
## Step 4: Download source code and install Keystone
	
    cd /<source_root>/
	git clone https://github.com/openstack/keystone.git
    cd keystone/
    git checkout 9.2.0
    sudo pip install -r requirements.txt
    sudo pip install -r test-requirements.txt 
    sudo python setup.py install
	
## Step 5: Configure Keystone  

    sudo cp -r etc/ /etc/keystone
    cd /etc/keystone/
    sudo mv keystone.conf.sample keystone.conf
    sudo mv logging.conf.sample logging.conf
    sudo vi keystone.conf 

* Edit `keystone.conf` file as shown below

	```diff
	@@ -1,4 +1,6 @@
	[DEFAULT]
	+	    verbose = True
	+	    admin_token=ADMIN
	#
	# From keystone
	#

	@@ -1769,7 +1772,7 @@
	# Entrypoint for an implementation of the backend for persisting revocation
	# events in the keystone.revoke namespace. Supplied drivers are kvs and sql.
	# (string value)
	-    #driver = sql
	+    driver = sql

	# This value (calculated in seconds) is added to token expiration before a
	# revocation event may be removed from the backend. (integer value)

	@@ -546,6 +548,7 @@
	# Deprecated group/name - [DATABASE]/sql_connection
	# Deprecated group/name - [sql]/connection
	#connection = <None>
	+    connection = mysql://keystone:<KEYSTONE_DBPASS>@<DB_HOST>/keystone

	# The SQLAlchemy connection string to use to connect to the slave database.
	# (string value)

	@@ -2001,12 +2004,12 @@
	# Controls the token construction, validation, and revocation operations.
	# Entrypoint in the keystone.token.provider namespace. Core providers are
	# [fernet|pkiz|pki|uuid]. (string value)
	-    #provider = uuid
	+    provider = uuid

	# Entrypoint for the token persistence backend driver in the
	# keystone.token.persistence namespace. Supplied drivers are kvs, memcache,
	# memcache_pool, and sql. (string value)
	-    #driver = sql
	+    driver = sql

	# Toggle for token system caching. This has no effect unless global caching is
	# enabled. (boolean value)

	@@ -1248,7 +1251,7 @@
	#

	# Memcache servers in the format of "host:port". (list value)
	-    #servers = localhost:11211
	+    servers = localhost:11211

	# Number of seconds memcached server is considered dead before it is tried
	# again. This is used by the key value store system (e.g. token pooled
	```      

* Populate Keystone database  

	```
	keystone-manage db_sync 
	``` 
	
_**Note**_: Value of `admin_token` in [default] section can be any random string which is used later
    
## Step 6: Start Keystone service

Follow below instructions to enable wsgi to serve Keystone requests

#### Edit httpd.conf

* RHEL 7.2/7.3 

	* Add below content at end of **/etc/httpd/conf/httpd.conf** file:
	
        ```
        ServerName <KEYSTONE_HOST_IP>
        Include /etc/httpd/sites-enabled/	
        ```                

* SLES 12-SP1/12-SP2  

	* Add below content at end of **/etc/apache2/httpd.conf** file:
	
        ```
        ServerName <KEYSTONE_HOST_IP>
        Include /etc/apache2/sites-enabled/
        LoadModule wsgi_module /usr/lib64/python2.7/site-packages/mod_wsgi/server/mod_wsgi-py27.so	
        ```        
	
    _**Note**_: Comment out the below line in **/etc/apache2/httpd.conf** file if it exist:

		Include /etc/apache2/sysconfig.d/include.conf

* Ubuntu 16.04

    * Add below content at end of **/etc/apache2/apache2.conf** file:
    
        ```
        ServerName <KEYSTONE_HOST_IP>
        ```        

#### Add wsgi-keystone.conf
wsgi-keystone.conf content is different on RHEL 7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04 due to difference in apache installation

* RHEL 7.2/7.3  

	```
	cd /etc/httpd/
	sudo mkdir sites-available
	sudo mkdir sites-enabled
	cd sites-available/
	sudo vi wsgi-keystone.conf
	```

	Add following Content to the file
	```
	 Listen 5000
	 Listen 35357
	 <VirtualHost *:5000>  
	     WSGIDaemonProcess keystone-public processes=5 threads=1  
	     WSGIProcessGroup keystone-public  
	     WSGIScriptAlias / /var/www/cgi-bin/keystone/main  
	     WSGIApplicationGroup %{GLOBAL}  
	     WSGIPassAuthorization On  
	     <IfVersion >= 2.4>  
	         ErrorLogFormat "%{cu}t %M"  
	     </IfVersion>  
	     LogLevel info  
	     ErrorLog /var/log/httpd/keystone-error.log  
	     CustomLog /var/log/httpd/keystone-access.log combined  
	 </VirtualHost>  

	 <VirtualHost *:35357>
	     WSGIDaemonProcess keystone-admin processes=5 threads=1  
	     WSGIProcessGroup keystone-admin  
	     WSGIScriptAlias / /var/www/cgi-bin/keystone/admin  
	     WSGIApplicationGroup %{GLOBAL}  
	     WSGIPassAuthorization On  
	     <IfVersion >= 2.4>  
	         ErrorLogFormat "%{cu}t %M"  
	     </IfVersion>  
	     LogLevel info  
	     ErrorLog /var/log/httpd/keystone-error.log  
	     CustomLog /var/log/httpd/keystone-access.log combined  
	 </VirtualHost>		
	```
* SLES 12-SP1/12-SP2 

	```
	cd /etc/apache2/
	sudo mkdir sites-available
	sudo mkdir sites-enabled
	cd sites-available/
	sudo vi wsgi-keystone.conf
	```
	
	Add following Content to the file
	```diff
	 Listen 5000
	 Listen 35357
	 <VirtualHost *:5000>  
	     WSGIDaemonProcess keystone-public processes=5 threads=1  
	     WSGIProcessGroup keystone-public  
	     WSGIScriptAlias / /var/www/cgi-bin/keystone/main  
	     WSGIApplicationGroup %{GLOBAL}  
	     WSGIPassAuthorization On  
	     LogLevel info  
	     ErrorLog /var/log/apache2/keystone-error.log  
	     CustomLog /var/log/apache2/keystone-access.log combined  

	     <Directory /var/www/cgi-bin/keystone>  
	         Options Indexes FollowSymLinks MultiViews  
		  AllowOverride all  
		  Require all granted  
	     </Directory>  
	 </VirtualHost>  

	 <VirtualHost *:35357>  
	     WSGIDaemonProcess keystone-admin processes=5 threads=1  
	     WSGIProcessGroup keystone-admin  
	     WSGIScriptAlias / /var/www/cgi-bin/keystone/admin  
	     WSGIApplicationGroup %{GLOBAL}  
	     WSGIPassAuthorization On  
	     LogLevel info  
	     ErrorLog /var/log/apache2/keystone-error.log  
	     CustomLog /var/log/apache2/keystone-access.log combined  
	       			
	     <Directory /var/www/cgi-bin/keystone>  
	         Options Indexes FollowSymLinks MultiViews  
		  AllowOverride all  
		  Require all granted  
	      </Directory>  
	 </VirtualHost>
	```

* Ubuntu 16.04

	```
	cd /etc/apache2/
	sudo mkdir -p sites-available
	sudo mkdir -p sites-enabled
	cd sites-available/
	sudo vi wsgi-keystone.conf
	```
	
	Add following Content to the file
	```diff
	 Listen 5000
	 Listen 35357

	 <VirtualHost *:5000>
	     WSGIDaemonProcess keystone-public processes=5 threads=1  
	     WSGIProcessGroup keystone-public  
	     WSGIScriptAlias / /var/www/cgi-bin/keystone/main  
	     WSGIApplicationGroup %{GLOBAL}  
	     WSGIPassAuthorization On  
	     <IfVersion >= 2.4>  
	         ErrorLogFormat "%{cu}t %M"  
	     </IfVersion>  
	     LogLevel info  
	     ErrorLog /var/log/apache2/keystone-error.log  
	     CustomLog /var/log/apache2/keystone-access.log combined  
	 </VirtualHost>  

	 <VirtualHost *:35357>
	     WSGIDaemonProcess keystone-admin processes=5 threads=1  
	     WSGIProcessGroup keystone-admin  
	     WSGIScriptAlias / /var/www/cgi-bin/keystone/admin  
	     WSGIApplicationGroup %{GLOBAL}  
	     WSGIPassAuthorization On  
	     <IfVersion >= 2.4>  
	         ErrorLogFormat "%{cu}t %M"  
	     </IfVersion>  
	     LogLevel info  
	     ErrorLog /var/log/apache2/keystone-error.log  
	     CustomLog /var/log/apache2/keystone-access.log combined  
	 </VirtualHost>  
	```	
	
Enable the Identity service virtual host

* RHEL 7.2/7.3 

	```
	sudo ln -s /etc/httpd/sites-available/wsgi-keystone.conf /etc/httpd/sites-enabled 
	```

* SLES 12-SP1/12-SP2, Ubuntu 16.04

	```
	sudo ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
	```

Create the directory structure for the WSGI components 

    sudo mkdir -p /var/www/cgi-bin/keystone
	
Copy the WSGI components from the upstream repository into this directory 

    curl http://git.openstack.org/cgit/openstack/keystone/plain/httpd/keystone.py?h=stable/mitaka | sudo tee /var/www/cgi-bin/keystone/main /var/www/cgi-bin/keystone/admin 
	
Adjust permissions on this directory and the files in it 

    sudo chmod 755 /var/www/cgi-bin/keystone/* 
	
Start apache service 

* RHEL 7.2/7.3, SLES 12-SP1/12-SP2 

	```
	sudo /usr/sbin/httpd 
	```
	
* Ubuntu 16.04

	```
	sudo service apache2 start 
	```
	
_**Note**_: 

* This command internally starts Keystone service  
* Comment ulimit section if required, in file `/usr/sbin/apache2ctl` and restart apache
  
## Step 7: Create the service entity and API endpoint
1. Export **os-token** (which is nothing but admin_token, set in keystone.conf), **os-endpoint** 

	```
	export OS_SERVICE_TOKEN=ADMIN
	export OS_SERVICE_ENDPOINT=http://<KEYSTONE_HOST_IP>:35357/v2.0/
	```
	
2. Steps to create endpoints and service 

	```
	keystone service-create --name=keystone --type=identity --description="Keystone Identity Service" 	
	keystone endpoint-create --service=keystone --publicurl=http://<KEYSTONE_HOST_IP>:5000/v2.0 --internalurl=http://<KEYSTONE_HOST_IP>:5000/v2.0 --adminurl=http://<KEYSTONE_HOST_IP>:35357/v2.0
	```
	
## Step 8: Create projects, users, and roles

The identity service provides authentication services for each OpenStack service. The authentication service uses a combination of tenants, users, and roles 

    keystone tenant-create --name=admin
    keystone tenant-create --name=service --description="Service Tenant"
    keystone user-create --name=admin --tenant=admin --pass=ADMIN
    keystone role-create --name=admin
    keystone user-role-add --user=admin --tenant=admin --role=admin
	
### Step 9: Verify Keystone installation  

    unset OS_SERVICE_TOKEN OS_SERVICE_ENDPOINT
    export OS_USERNAME=admin
    export OS_PASSWORD=ADMIN
    export OS_TENANT_NAME=admin
    export OS_AUTH_URL=http://<KEYSTONE_HOST_IP>:5000/v2.0
	
Run any Keystone command and check if it succeeds. For example 

    keystone token-get

### References:
http://docs.openstack.org/kilo/install-guide/install/apt/content/ch_keystone.html
http://docs.openstack.org/developer/keystone/installing.html