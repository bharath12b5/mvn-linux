The [Open Source Puppet 4.1.0] (https://puppetlabs.com/puppet/puppet-open-source) can be built on RHEL7, RHEL 6, SLES 11 and SLES 12 on IBM z System by following these instructions.

## Puppet master installation

1. Install Ruby and RubyGems

    ```
    $ wget http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz
    $ tar -xvf ruby-2.2.2.tar.gz
    $ cd ruby-2.2.2
    $ ./configure
    $ make
    $ make install
    ```

    If you do not have wget installed, perform

    `$yum install wget` for RHEL or

    `$zypper install wget` for SLES

2. Edit the environment variable:

    ```$ vi ˜/.bash_profile```

    append `PATH=$PATH:/usr/local/bin`, and save.

3. Install Puppet

    ```$ gem install puppet```

4. Locate the `$confdir` by command

    ```$ puppet master --configprint confdir```

    and the output gives the directory. If such directory does not exit, create one. For example, if the output is `/etc/puppetlabs/puppet`, then

    ```$ mkdir -p /etc/puppetlabs/puppet```

5. Create necessary directories and files in `$confdir`

    ```
    $ puppet master --genconfig > /etc/puppetlabs/puppet/puppet.conf
    $ mkdir /etc/puppetlabs/puppet/modules
    $ mkdir /etc/puppetlabs/puppet/manifests
    $ cd /etc/puppetlabs/puppet
    $ wget https://raw.githubusercontent.com/puppetlabs/puppet/master/conf/auth.conf
    ```

6. Create other necessary directories

    ```
    $ mkdir -p /opt/puppetlabs/puppet
    $ mkdir -p /var/log/puppetlabs
    ```

7. Comment the "configtimeout" setting in `$confdir/puppet.conf`

    ```$ vi /etc/puppetlabs/puppet/puppet.conf```

    in vi, type "/configtimeout", press "n" to locate the setting "configtimeout" and input a "#" in front of it to comment it.

8. Create a group and user for the master:

    ```
    $ puppet resource group puppet ensure=present
    $ puppet resource user puppet ensure=present gid=puppet shell='/sbin/nologin'
    ```

9. Modify the `$confdir/puppet.conf`. If we assume the hostname of the master machine is master.myhost.com, change the values of servers (either in the main or master section) as follows:

    ```
    server=master.myhost.com
    ca_server=$server
    report_server=$server
    archive_file_server=$server
    ```

    make sure the values of all servers are DNS resolvable. Here we have assigned all the servers to be master.myhost.com, they could also be different servers.

10. The Puppet master runs on TCP port 8140. This port needs to be open on your master’s firewall (and any intervening firewalls and network devices), and your agent must be able to route and connect to the master. To do this, you need to have an appropriate firewall rule on your master, such as the following rule for the Netfilter firewall:

    ```$ iptables -A INPUT -p tcp -m state --state NEW --dport 8140 -j ACCEPT```

## Puppet agent installation

1. Install Ruby and RubyGems

    ```
    $ wget http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz
    $ tar -xvf ruby-2.2.2.tar.gz
    $ cd ruby-2.2.2
    $ ./configure
    $ make
    $ make install
    ```

    If you do not have wget installed, perform

    `$yum install wget` for RHEL or

    `$zypper install wget` for SLES

2. Edit the environment variable:

    ```$ vi ˜/.bash_profile```

    append `PATH=$PATH:/usr/local/bin`, and save.

3. Install Puppet

    ```$ gem install puppet```

4. Locate the `$confdir` by command

    ```$ puppet agent --configprint confdir```

    if such directory does not exit, create one. For example, if the output is `/etc/puppetlabs/puppet`, then

    ```$ mkdir -p /etc/puppetlabs/puppet```

5. Create necessary directories and files in `$confdir`

    ```
    $ puppet agent --genconfig > /etc/puppetlabs/puppet/puppet.conf
    $ cd /etc/puppetlabs/puppet
    ```

6. Create other necessary directories

    ```
    $ mkdir -p /opt/puppetlabs/puppet
    $ mkdir -p /var/log/puppetlabs
    ```

7. Comment the "configtimeout" setting in `$confdir/puppet.conf`

    ````
    $ vi /etc/puppetlabs/puppet/puppet.conf
    ```

    in vi, type "/configtimeout", press "n" to locate the setting "configtimeout" and input a "#" in front of it to comment it.

8. Modify the `$confdir/puppet.conf`. If we assume the hostname of the master machine is master.myhost.com, change the values of servers (servers in the main or agent section) as follows:

    ```
    server=master.myhost.com
    ca_server=$server
    report_server=$server
    archive_file_server=$server
    ```

    make sure the values of all servers are DNS resolvable to the agent machine. Here we have assigned all the servers to be master.myhost.com, they could also be different servers.

9. Puppet runs on TCP port 8140.

    ```$ iptables -A INPUT -p tcp -m state --state NEW --dport 8140 -j ACCEPT```

## Connecting the Master and Agent for the first time

1. On the master machine (assuming with hostname master.myhost.com), run the master application:

    ```$ puppet master --verbose --no-daemonize```

    The --verbose option outputs verbose logging and the --no-daemonize option keeps the daemon in the foreground and redirects output to standard out. You can also add the --debug option to produce more verbose debug output from the daemon.

  On the agent application (assuming the hostname of the agent is agent.myhost.com):

    ```$ puppet agent --test```

    You can see the output from our connection. The agent has created a certificate signing request and a private key to secure our connection. Puppet uses SSL certificates to authenticate connections between the master and the agent. The agent sends the certificate request to the master and waits for the master to sign and return the certificate. At this point, the agent has exited after sending in its Certificate Signing Request (CSR). The agent will need to be rerun to check in and run Puppet after the CSR has been signed by the CA. You can configure Puppet agent not to exit, but instead stay alive and poll periodically for the CSR to be signed. This configuration is called waitforcert and is generally only useful if you are also auto-signing certificates on the master.

2. On the master machine, to complete the connection and authenticate our agent, we now need to sign the certificate the agent has sent to the master. We do this using puppet cert (or the puppetca binary) on the master:

    ```$ puppet cert list```

    The list option displays all the certificates waiting to be signed. We can then sign our certificate using the sign option:

    ```$ puppet cert sign agent.myhost.com```

3. Restart both master and agent, now they should connect.

4. If an error occurs, showing that

    > Info: Retrieving pluginfacts

    > Error: /File[/opt/puppetlabs/puppet/cache/facts.d]: Could not evaluate: Could not retrieve information from environment production source(s) puppet:///pluginfacts

    > Info: Retrieving plugin

    > Error: /File[/opt/puppetlabs/puppet/cache/lib]: Could not evaluate: Could not retrieve information from environment production source(s) puppet://master.myhost.com/plugins

    This is because you don't have any plugins to syn yet, and the pluginsyn property is set to be true by default. So solutions are:

    1). Disable the setting in the agent's config file by setting `pluginsyn=false`. Or
    2). Create at least one plugin.

## Testing
For testing, we run the tests from the source code

1. Download  Puppet source code and install bundler

    ```
    $ git clone git://github.com/puppetlabs/puppet
    $ cd puppet
    $ gem install bundler
    $ gem update
    $ bundle init
    $ bundle install
    ```

2. Run the test

    ``` $ bundle exec rspec spec```

[One known test error specific to z systems reported] (https://tickets.puppetlabs.com/browse/PUP-4962)

## Reference
1. https://docs.puppetlabs.com/guides/install_puppet/pre_install.html