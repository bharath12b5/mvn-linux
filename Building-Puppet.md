<!---PACKAGE:Puppet--->
<!---DISTRO:SLES 12:4.5.3--->
<!---DISTRO:SLES 11:4.5.3--->
<!---DISTRO:RHEL 7.1:4.5.3--->
<!---DISTRO:RHEL 6.6:4.5.3--->
<!---DISTRO:Ubuntu 16.x:Distro, 4.5.3--->


# Building Puppet

Below versions of Puppet are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `3.8.5`

The instructions provided below specify the steps to build Puppet version 4.5.3 on Linux on the IBM z Systems for RHEL 6/7, SLES 11/12 and Ubuntu 16.04.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Puppet Master Installation
1. Install following dependencies  
  
   For RHEL6 & RHEL7:  
    ```
    sudo yum install -y gcc-c++ readline-devel tar openssl unzip libyaml-devel PackageKit-cron openssl-devel make git wget sqlite-devel glibc-common
```
   For SLES11 & SLES12:  
    ````
    sudo zypper install -y gcc-c++ readline-devel tar openssl unzip openssl-devel make git wget sqlite-devel glibc-locale
````
   For Ubuntu 16.04:  
    ````
    sudo apt-get install -y g++ libreadline6 libreadline6-dev tar openssl unzip libyaml-dev libssl-dev make git wget libsqlite3-dev  libc6-dev cron
````

2. Download and install Ruby
    ````
    cd /<source_root>/
    wget http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz
    tar -xvf ruby-2.2.2.tar.gz
    cd ruby-2.2.2   
	./configure && make && sudo make install
````

3. Download and install RubyGems
    ````
    cd /<source_root>/
    wget http://production.cf.rubygems.org/rubygems/rubygems-2.2.2.tgz
    tar -xvf rubygems-2.2.2.tgz
    cd rubygems-2.2.2
	sudo /usr/local/bin/ruby setup.rb                
````

4. Install bundler
    ````
    cd /<source_root>/
    sudo /usr/local/bin/gem install bundler rake-compiler	
````

5. Install Puppet
    ````
    cd /<source_root>/
    sudo /usr/local/bin/gem install puppet -v 4.5.3	
````

6. Locate the  $confdir  by command  
**Note:** *Please make sure the directory `/usr/local/lib` has sufficient read and execute permissions to run the above command.Incase it doesn't,run`sudo chown 755 /usr/local/lib` to give the necessary permissions* 
    ````
    confdir=`puppet master --configprint confdir`
	echo $confdir
````
The output gives the directory. If such directory does not exist, create one. 
    ````	
    mkdir -p $confdir
````

7. Create necessary directories and files in  $confdir
    ````
    mkdir $confdir/modules
    mkdir $confdir/manifests
    cd $confdir
    touch puppet.conf
    wget https://raw.githubusercontent.com/puppetlabs/puppet/master/conf/auth.conf
    mkdir -p $confdir/opt/puppetlabs/puppet
    mkdir -p $confdir/var/log/puppetlabs
````

9. Create "puppet" user and group
    ````
    sudo useradd -d /home/puppet -m -s /bin/bash puppet
    sudo /usr/local/bin/puppet resource group puppet ensure=present
````
```Note```: Set a user specified password for puppet user .Running **sudo passwd puppet** will prompt for new password
    


10. Add the following parameters to $confdir/puppet.conf (assuming hostname of the master machine is master.myhost.com)    
**Note:** *Hostname can found by running the command `hostname -f`* 
    ````
    [main]
          logdir = $confdir/var/log/puppetlabs
          basemodulepath = $confdir/modules
          server = master.myhost.com
          user  = puppet
          group = puppet
          pluginsync = false
     [master]
          certname = master.myhost.com
          autosign = true
````

11. The Puppet master runs on TCP port 8140. This port needs to be open on your master’s firewall (and any intervening firewalls and network devices), and your agent must be able to route and connect to the master. To do this, you need to have an appropriate firewall rule on your master, such as the following rule for the Netfilter firewall
    ````
    sudo iptables -A INPUT -p tcp -m state --state NEW --dport 8140 -j ACCEPT 
````

## Puppet Agent Installation
1. Install following dependencies
  
  For RHEL6 & RHEL7:  

    ````
    sudo yum install -y gcc-c++ readline-devel tar openssl unzip libyaml-devel PackageKit-cron openssl-devel make git wget sqlite-devel glibc-common
````
  For SLES11 & SLES12:
    ````
    sudo zypper install -y gcc-c++ readline-devel tar openssl unzip openssl-devel make git wget sqlite-devel glibc-locale
````
  For Ubuntu 16.04:  
    ````
    sudo apt-get install -y g++ libreadline6 libreadline6-dev tar openssl unzip libyaml-dev libssl-dev make git wget libsqlite3-dev  libc6-dev
````

2. Download and install Ruby
    ````
    cd /<source_root>/
    wget http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz
    tar -xvf ruby-2.2.2.tar.gz
    cd ruby-2.2.2
    ./configure && make && sudo make install
````

3. Download and install RubyGems
    ````
     cd /<source_root>/
     wget http://production.cf.rubygems.org/rubygems/rubygems-2.2.2.tgz
     tar -xvf rubygems-2.2.2.tgz
     cd rubygems-2.2.2
     sudo /usr/local/bin/ruby  setup.rb
````

4. Install bundler
    ````
    cd /<source_root>/
    sudo /usr/local/bin/gem install bundler rake-compiler
````

5. Install Puppet
    ````
    cd /<source_root>/
    sudo /usr/local/bin/gem install puppet -v 4.5.3
````

6. Locate the  $confdir  by command    
**Note:** *Please make sure the directory `/usr/local/lib` has sufficent read and execute permissions to run the above command.Incase it doesn't,run `sudo chown 755 /usr/local/lib` to give the necessary permissions* 
    ````
    confdir=`puppet agent --configprint confdir`
	echo $confdir
````
The output gives the directory. If such directory does not exist, create one. 
    ````	
    mkdir -p $confdir
````

7. Create necessary directories and files in  $confdir
    ````
    cd $confdir
    mkdir -p $confdir/opt/puppetlabs/puppet
    mkdir -p $confdir/var/log/puppetlabs
    touch puppet.conf
````

8. Add the following parameters to $confdir/puppet.conf (assuming hostname of the master machine is master.myhost.com and hostname of the agent machine is agent.myhost.com)    
**Note:** *Hostname can found by running the command `hostname -f`* 
    ````
    [main]
          logdir =  $confdir/var/log/puppetlabs
          basemodulepath = /etc/puppetlabs/puppet/modules
          server = master.myhost.com
          user  = puppet
          group = puppet
          pluginsync = false
     [agent]
          certname = agent.myhost.com
          report = true
          pluginsync = false
````

9. Add an entry in /etc/hosts file with ipaddress and hostname of master node
    ````
     sudo vi /etc/hosts
     <master ipaddress> <master hostname>
````

## Connecting the Master and Agent for the first time

1. Run the master application on master machine (assuming with hostname master.myhost.com)
    ````
    puppet master --verbose --no-daemonize 
````
The --verbose option outputs verbose logging and the --no-daemonize option keeps the daemon in the foreground and redirects output to standard output. You can also add the --debug option to produce more verbose debug output from the daemon.

2. On the agent application (assuming the hostname of the agent is agent.myhost.com)
    ````
    puppet agent --test 
````
Note: The following errors might be seen after execution of the above step

    Info: Retrieving pluginfacts
    Error: /File[/opt/puppetlabs/puppet/cache/facts.d]: Could not evaluate: Could not retrieve information from environment production source(s) puppet:///pluginfacts
    Info: Retrieving plugin
    Error: /File[/opt/puppetlabs/puppet/cache/lib]: Could not evaluate: Could not retrieve information from environment production source(s) puppet://master.myhost.com/plugins

    This is because you don't have any plugins to syn yet, and the pluginsyn property is set to be true by default. So solutions are:

	1) Disable the setting in the agent's 'puppet.conf' file by setting  pluginsyn=false. Or 

	2) Create at least one plugin ( By running puppet module install saz-sudo  on master)

## Testing  
For testing, run the tests from the source code on Master machine. 
 
1. Switch user to puppet, clone Puppet git repository in /home/puppet and execute "bundle install" to install the required gems    
    ````
     su puppet
     cd /home/puppet
     git clone --branch 4.5.3 git://github.com/puppetlabs/puppet
     cd puppet
     bundle install --path .bundle/gems/
````

2. Edit file_spec.rb to support the testcases in environment  
    ````
    cd /home/puppet/puppet
````  
Change the below line at number 59 in ```spec/unit/indirector/file_bucket_file/file_spec.rb  ```
    ````
    end.to raise_error(Puppet::FileBucket::BucketError, /Got passed new contents/)
````
Replace it with  
    ````
    should compile.and_raise_error(Puppet::FileBucket::BucketError, /Got passed new contents/)
        end
````  
3. Running the test cases  

	Few testcases need to be executed as root user and others as puppet user.  

	3.1. Execute testcases as root user:  
  
    _* Unit testcases, except ssl, face, indirector, network related testcases, should be executed as root user._  
    _* The integration testcases for provider and type should be executed as root user._  
    _* ```Note```: Run the  below commands as root user.You can switch to root user by running **exit**, if you are currently switched to puppet user._  

    1. Create a shell script   
    For example,` rootuser_tests.sh `

        ````
    cd /home/puppet/puppet  
    touch rootuser_tests.sh  
    chmod +x rootuser_tests.sh  
    ````

    2. Add the following content to the shell script         
        ````
    #!/bin/bash
    set -e
        
    echo "Running Unit testcases as root user"
    declare -a unittests1
    unittests1=$(ls spec/unit|egrep -v "ssl|face|indirector|network")
    unittest_list1=($unittests1)
    for i in "${unittest_list1[@]}"
    do
	  bundle exec rspec "spec/unit/$i"
    done
        
    echo "Running Integration testcases as root user"
    declare -a integration1
    integration1=$(ls spec/integration|egrep "provider|type")
    integration_list1=($integration1)
    for j in "${integration_list1[@]}"
    do
      bundle exec rspec --exclude-pattern ./spec/integration/provider/service/systemd_spec.rb "spec/integration/$j"
    done
    ````

    3. Run the shell script
 
      * For Ubuntu 16.04

          ```
          locale-gen "en_US.UTF-8"
          ./rootuser_tests.sh
          ```
      * For RHEL 6/RHEL 7/SLES 11/SLES 12

          ```
          export LC_ALL="en_US.UTF8"
          ./rootuser_tests.sh
          ```

	3.2. Execute testcases as puppet user  
	
    _* ssl, face, indirector, network related  unit testcases should be executed as puppet user._  
    _* The integration testcases except provider and type related testcases should be executed as puppet user._  
    _* data_binding.rb file is not executed as it does not involve any testcases to be invoked directly._  
    
	1. Create a shell script  
    For example `puppetuser_tests.sh` 
        ````
    cd /home/puppet/puppet
    touch puppetuser_tests.sh
    chmod +x puppetuser_tests.sh
    ````

    2. Add the following content to the script  
        ````
    #!/bin/bash
    set -e
        
    echo "Running Unit testcases as puppet user"
    declare -a unittests2
    unittests2=$(ls spec/unit|egrep "ssl|face|indirector|network")
    unittest_list2=($unittests2)
    for i in "${unittest_list2[@]}"
    do
      bundle exec rspec "spec/unit/$i"
    done
        
    echo "Running Integration testcases as puppet user"
    declare -a integration2
    integration2=$(ls spec/integration|egrep -v "data_binding.rb|provider|type")
    integration_list2=($integration2)
    for j in "${integration_list2[@]}"
    do
      bundle exec rspec "spec/integration/$j"
    done
    ````

    3. Switch user to puppet  
        ````
    su puppet
    ````

    4. Run the shell script  
        ````
    ./puppetuser_tests.sh
    ````
    
## References:
    [https://puppetlabs.com/](https://puppetlabs.com/)