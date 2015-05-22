The [Open Source Puppet 4.0.0] (https://puppetlabs.com/puppet/puppet-open-source) can be built on RHEL7/ RHEL 6 on IBM z System by following these instructions.

## Puppet master installation

1. Install ruby and ruby gem

    ```
    $ wget http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz
    $ tar -xvf ruby-2.2.2.tar.gz
    $ cd ruby-2.2.2
    $ ./configure
    $ make
    $ make install
    ```

    If you do not have wget installed, perform

    `$yum install wget`

2. Edit the environment variable:

    ```$ vi ˜/.bash_profile```

    append `PATH=$PATH:/usr/local/bin`, and save.

3. Install Puppet

    ```$ gem install puppet```

4. Locate the `$condir` by command

    ```$ puppet master --configprint confdir```

    and the output gives the directory. If such directory does not exit, create one. For example, if the output is `/etc/puppetlabs/puppet`, then

    ```$ mkdir -p /etc/puppetlabs/puppet```

5. Create necessary directories and files in `$condir`

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

7. Comment the "configtimeout" setting in `$condir/puppet.conf`

    ```$ vi /etc/puppetlabs/puppet/puppet.conf```

    in vi, type "/configtimeout", press "n" to locate the setting "configtimeout" and input a "#" in front of it to comment it.

8. Create a group and user for the master:

    ```
    $ puppet resource group puppet ensure=present
    $ puppet resource user puppet ensure=present gid=puppet shell='/sbin/nologin'
    ```

9. Modify the `$condir/puppet.conf`. If we assume the hostname of the master machine is master.ibm.com, change the values of dns_alt/_names and server (both in the main or master section) as follows:

    ```
    dns_alt_names = puppet,master.ibm.com
    server=master.ibm.com
    ```

    make sure the value of server is the hostname of the master machine and it is DNS resolvable to the agent machine.

10. The Puppet master runs on TCP port 8140. This port needs to be open on your master’s firewall (and any intervening firewalls and network devices), and your client must be able to route and connect to the master. To do this, you need to have an appropriate firewall rule on your master, such as the following rule for the Netfilter firewall:

    ```$ iptables -A INPUT -p tcp -m state --state NEW --dport 8140 -j ACCEPT```

## Puppet agent installation

1. Install ruby and ruby gem

    ```
    $ wget http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz
    $ tar -xvf ruby-2.2.2.tar.gz
    $ cd ruby-2.2.2
    $ ./configure
    $ make
    $ make install
    ```

    If you do not have wget installed, perform

    ```$yum install wget```

2. Edit the environment variable:

    ```$ vi ˜/.bash_profile```

    append `PATH=$PATH:/usr/local/bin`, and save.

3. Install Puppet

    ```$ gem install puppet```

4. Locate the `$condir` by command

    ```$ puppet agent --configprint confdir```

    if such directory does not exit, create one. For example, if the output is `/etc/puppetlabs/puppet`, then

    ```$ mkdir -p /etc/puppetlabs/puppet```

5. Create necessary directories and files in `$condir`

    ```
    $ puppet agent --genconfig > /etc/puppetlabs/puppet/puppet.conf
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

7. Comment the "configtimeout" setting in `$condir/puppet.conf`

    ````
    $ vi /etc/puppetlabs/puppet/puppet.conf
    ```

    in vi, type "/configtimeout", press "n" to locate the setting "configtimeout" and input a "#" in front of it to comment it.

8. Modify the `$condir/puppet.conf`. If we assume the hostname of the master machine is master.ibm.com, change the values of dns_alt_names and server (both in the main or master section) as follows:

    ```
    dns_alt_names = puppet,master.ibm.com
    server=master.ibm.com
    ```

    make sure the server is the hostname of the master machine and the agent hostname is DNS resolvable to the master machine.

9. Puppet runs on TCP port 8140.

    ```$ iptables -A INPUT -p tcp -m state --state NEW --dport 8140 -j ACCEPT```

## Connecting the Master and Agent for the first time

1. On the master machine (assuming with hostname master.ibm.com), run the master application:

    ```$ puppet master --verbose --no-daemonize```

    The --verbose option outputs verbose logging and the --no-daemonize option keeps the daemon in the foreground and redirects output to standard out. You can also add the --debug option to produce more verbose debug output from the daemon.

  On the agent application (assuming the hostname of the agent is agent.ibm.com):

    ```$ puppet agent --test```

    You can see the output from our connection. The agent has created a certificate signing request and a private key to secure our connection. Puppet uses SSL certificates to authenticate connections between the master and the agent. The agent sends the certificate request to the master and waits for the master to sign and return the certificate. At this point, the agent has exited after sending in its Certificate Signing Request (CSR). The agent will need to be rerun to check in and run Puppet after the CSR has been signed by the CA. You can configure Puppet agent not to exit, but instead stay alive and poll periodically for the CSR to be signed. This configuration is called waitforcert and is generally only useful if you are also auto-signing certificates on the master.

2. On the Master side. To complete the connection and authenticate our agent, we now need to sign the certificate the agent has sent to the master. We do this using puppet cert (or the puppetca binary) on the master:

    ```$ puppet cert list```

    The list option displays all the certificates waiting to be signed. We can then sign our certificate using the sign option:

    ```$ puppet cert sign agent.ibm.com```

3. Restart both master and client, now they should connect.

4. If an error occurs, showing that

    > Info: Retrieving pluginfacts

    > Error: /File[/opt/puppetlabs/puppet/cache/facts.d]: Could not evaluate: Could not retrieve information from environment production source(s) puppet:///pluginfacts

    > Info: Retrieving plugin

    > Error: /File[/opt/puppetlabs/puppet/cache/lib]: Could not evaluate: Could not retrieve information from environment production source(s) puppet://master.ibm.com/plugins

    This is because you don't have any plugins to syn yet, and the pluginsyn property is set to be true by default. So solutions are:

    1). Disable the setting in the agent's config file: pluginsyn=false. Or
    2). Create at least one plugin.

## Reference
1. https://docs.puppetlabs.com/guides/install_puppet/pre_install.html