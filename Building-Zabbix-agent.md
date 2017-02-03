## Building Zabbix agent

Below versions of Zabbix agent are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04/16.10 have `2.4.7`

The instructions provided below specify the steps to build Zabbix agent version 3.2.1 on the IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	 

* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._


##Step 1: Install dependencies  


* RHEL 6.8, RHEL 7.1/7.2/7.3 
 ```shell
 sudo yum install tar wget make gcc     
 ``` 

* SLES 11-SP4 and SLES 12/12-SP1/12-SP2
 ```shell   
 sudo zypper install tar wget make gcc    
 ```    
 
* Ubuntu 16.04/16.10      
 ```shell
 sudo apt-get install tar wget make gcc
 ```  

##Step 2: Download and install Zabbix agent  

 ```shell
 cd /<source_root>/
 wget http://sourceforge.net/projects/zabbix/files/ZABBIX%20Latest%20Stable/3.2.1/zabbix-3.2.1.tar.gz/download
 tar -xvf download (for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10)
 tar -xvf zabbix-3.2.1.tar.gz (for SLES 11-SP4)
 cd /<source_root>/zabbix-3.2.1
 ./configure --enable-agent
 make
 sudo make install
 ```



##Step 3: Post installation steps	

* Create a 'zabbix' user required to start Zabbix agent daemon
 ```shell
 sudo /usr/sbin/groupadd zabbix
 sudo /usr/sbin/useradd -g zabbix zabbix
 ```

* Edit Zabbix agent configuration file `/usr/local/etc/zabbix_agentd.conf`

 * Enter Zabbix server IP address and Hostname in configuration file  

* Start Zabbix agent  
 ```shell
 export PATH=$PATH:/usr/local/sbin/
 zabbix_agentd            
 ```	
_**Note:**_  If you get an error " cannot open "/tmp/zabbix_agentd.log" ", change file permissions of Zabbix agent log file and start Zabbix agent again

   ```shell
   sudo chmod 766 /tmp/zabbix_agentd.log
   ```

* Verify the installed Zabbix agent version with the following command:

 ```shell
 zabbix_get -s <host_ip> -k "agent.version"
 ```
   
#### Reference:
https://www.zabbix.com/documentation/3.2/manual/installation/ 