<!---PACKAGE:Consul--->
<!---DISTRO:SLES 12:0.5.2--->
<!---DISTRO:SLES 11:0.5.2--->
<!---DISTRO:RHEL 7.1:0.5.2--->
<!---DISTRO:RHEL 6.6:0.5.2--->

# Building Consul

Consul version 0.5.2 has been successfully built and tested for Linux on z Systems. The following instructions can be used for RHEL 6.6/7.1 and SLES 11/12.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

##### Step 1: Install Prerequisites
1. Prerequisites : 
  * GccGo 
  * Go  

  For RHEL6.6 and SLES11:
    * GccGo: Please refer [GccGo](https://github.com/linux-on-ibm-z/docs/wiki/Building-gccgo) recipe to install Go.

  For RHEL7.1 and SLES12:
    * Go: Please refer [Go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go) recipe to install Go.

     
2. Install the build dependencies
  * For RHEL6.6:
  ```
sudo yum install -y git unzip 
```

  * For SLES11:
  ```
 sudo zypper install -y git-core unzip 
```

  * For RHEL7.1:
  ```
sudo yum install -y make wget unzip which
```
  * For SLES12:
  ```
sudo yum install -y make wget unzip
```

3. Install other build dependency: **git >= 1.8.5**  
_Git version 1.8.5 or later is compatible with this Go version to build Consul. Update git version >= 1.8.5 by building it from source._

  1. _[Optional]_ Check the version of any existing git executable.
    ```
  which git
  $(which git) --version
```   
_**Note:** If above command gives git version lesser than 1.8.5, then follow below steps, else jump to Step no. 2 i.e. **'Export required go environment variables'**._  

  2. Install build dependencies for git
    ```
  sudo yum install -y asciidoc-8.6.8-5.el7.noarch openssl-devel curl-devel expat-devel perl-ExtUtils-MakeMaker gettext-devel tar
```  

  3. Download the git source code
    ```
  cd /<source_root>/
  git clone https://github.com/git/git.git
  cd /<source_root>/git/
  git checkout v1.8.5
```

  4. Build and Install git
    ```
  make prefix=<build-location> && sudo make prefix=<build-location> install 
```

  5. Export git binary path to PATH environment variable
    ```
  export PATH=<build-location>/bin:$PATH  
```

##### Step 2: Export required go environment variables
  ```
 export GOPATH=/<source_root>
```
For RHEL6.6 and SLES11,  
```
 export CGO_LDFLAGS="-L /lib64 -l pthread"
```

##### Step 3: Download Consul source code
```
 mkdir -p $GOPATH/src/github.com/hashicorp/
 cd $GOPATH/src/github.com/hashicorp/
 git clone https://github.com/hashicorp/consul.git
 cd $GOPATH/src/github.com/hashicorp/consul
 git checkout v0.5.2
```
##### Step 4: Build Consul
```
 make
```
##### Step 5: Download Consul Web UI
```
 mkdir $GOPATH/consul_web_ui
 cd $GOPATH/consul_web_ui
 wget https://releases.hashicorp.com/consul/0.5.2/consul_0.5.2_web_ui.zip
 unzip $GOPATH/consul_web_ui/consul_0.5.2_web_ui.zip
```
##### Step 6: Change working directory back to Consul
```
 cd $GOPATH/src/github.com/hashicorp/consul
```
##### Step 7: (Optional) Testing Consul
```
 make test
```

##### Step 8: Run a Consul server
_**Note:**_ To start a Consul server, you need to pass few mandatory configuration parameters while starting a server. It can be passed using _server.json_ file. The sample _server.json_ file looks like following :

```
{
  "datacenter": "<your-datacenter-name>",
  "data_dir": "/opt/consulserver",
  "log_level": "INFO",
  "node_name": "<your-server-node-name>",
  "server": true,
  "bootstrap": true,
  "ports" : {

    "dns" : -1,
    "http" : <http-port>,
    "rpc" : <RPC-port>,
    "serf_lan" : <LAN-port>,
    "serf_wan" : <WAN-port>,
    "server" : <internal-consul-port>
  }
}
```
_Click [here](https://www.consul.io/docs/agent/options.html) to read more about server configuration parameters_.

```
 sudo bin/consul agent -config-dir=/<path-to-server.json-file>/server.json
```
##### Step 9: Run a Consul agent
_Note:_ To start a Consul agent, you need to pass few mandatory configuration parameters while starting an agent. It can be passed using _agent.json_ file. The sample _agent.json_ file looks like following :

```
{
  "datacenter": "<your-datacenter-name>",
  "data_dir": "/opt/consulclient",
  "log_level": "INFO",
  "node_name": "<your-client-node-name>",
  "ui_dir": "<path-to-ui-dir-until-index.html>"
  "client_addr": "0.0.0.0",
  "ports" : {

    "dns" : -1,
    "http" : <http-port>,
    "rpc" : <RPC-port>,
    "serf_lan" : <LAN-port>,
    "serf_wan" : <WAN-port>,
    "server" : <internal-consul-port>
  },
  "start_join" : [
   "<consul-server-ip-address>:<LAN-port-of-consul-server>"
   ]
}
```
_Click [here](https://www.consul.io/docs/agent/options.html) to read more about agent configuration parameters_.

```
 sudo bin/consul agent -config-dir=/<path-to-server.json-file>/agent.json
```

##### Step 10: Access Consul web user interface from browser
```
 http://<consul-agent-host-ip>:<http-port>/ui/ 
```
_**Note:**_ Click [here](http://www.mammatustech.com/consul-service-discovery-and-health-for-microservices-architecture-tutorial) to read more about how to register a new service.

#### References:
https://www.consul.io/  
https://github.com/hashicorp/consul
