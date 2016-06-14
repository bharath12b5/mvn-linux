<!---PACKAGE:Consul--->
<!---DISTRO:SLES 12:0.6.4--->
<!---DISTRO:RHEL 7.1:0.6.4--->
<!---DISTRO:Ubuntu 16.x:0.6.4--->

The instructions provided below specify the steps to build Consul v0.6.4 on Linux on the IBM z Systems for RHEL 7.1, SLES 12 and Ubuntu 16.04.

_**General Notes:**_ 	 
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._
## Install Dependencies

For RHEL 7.1
   ```
   sudo yum install git vi gcc tar wget make unzip
   ```
   
For SLES 12
   ```
   sudo zypper install git vi gcc tar wget make unzip
   ```
   
For Ubuntu 16.04
   ```
   sudo apt-get update && \
   sudo apt-get install git vim gcc tar wget unzip
   ```
   
## Install GO Dependencies
   For RHEL 7.1 and SLES 12, refer [Go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go) recipe to install Go.

   For Ubuntu 16.04, install following dependency:    
   
   ```
      sudo apt-get install golang-1.6
   ```

## Building Consul from its source
1. Set GOPATH Environment Variables
	 
    ```
     export GOPATH=/<source_root>
    ```
2. Make directory & clone the source code
   ```
   mkdir $GOPATH/src
   cd $GOPATH/src
   mkdir -p ./github.com/hashicorp
   cd $GOPATH/src/github.com/hashicorp
   git clone --branch=v0.6.4 https://github.com/hashicorp/consul.git
   ```
   

3. Modify file `$GOPATH/src/github.com/hashicorp/consul/scripts/build.sh` with your choice
   of editor and replace lines from 40 to 45 with following commands:
   
  ```
    cd $DIR
    go install
   ```

   These lines invoke tool [gox](https://github.com/mitchellh/gox) and compile for
   different ARCH and OS combos.Unfortunately, [gox](https://github.com/mitchellh/gox)
   does not support s390x arch for now.Therefore, in order to build consul for z System,
   it is required to use the native build tool of go.
   
4. Replace sys package

   ```
    cd $GOPATH/src/github.com/hashicorp/consul/vendor/golang.org/x
    mv sys sys.bak
    git clone https://github.com/linux-on-ibm-z/sys.git
   ```
5. Build consul

   For RHEL 7.1 and SLES 12
   ```
   cd $GOPATH/src/github.com/hashicorp/consul/
   make
   ```
     
   For Ubuntu 16.04
   ```
   cd $GOPATH/src/github.com/hashicorp/consul/
   sudo -E make
   ```
6. Verify build success
   
   ```
   cd $GOPATH/bin
   ./consul
   ```

7. (Optional) Testing Consul  

   ```
   make test
   ```

8. Download Consul Web UI
   ```
   mkdir $GOPATH/consul_web_ui
   cd $GOPATH/consul_web_ui
   wget https://releases.hashicorp.com/consul/0.6.4/consul_0.6.4_web_ui.zip
   unzip $GOPATH/consul_web_ui/consul_0.6.4_web_ui.zip
   ```

9. Run Consul Server

  _**Note:**_ To start a Consul server, you need to pass few mandatory configuration parameters while starting a server.      It can be passed using _server.json_ file. The sample _server.json_ file looks like following :

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
     sudo $GOPATH/bin/consul agent -config-dir=/<path-to-server.json-file>/server.json
   ```

10. Run Consul Agent

  _Note:_ To start a Consul agent, you need to pass few mandatory configuration parameters while starting an agent. It can   be passed using _agent.json_ file. The sample _agent.json_ file looks like following :

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
   sudo $GOPATH/bin/consul agent -config-dir=/<path-to-server.json-file>/agent.json
  ```

11. Access Consul web user interface from browser
  ```
   http://<consul-agent-host-ip>:<http-port>/ui/ 
  ```
_**Note:**_ Click [here](http://www.mammatustech.com/consul-service-discovery-and-health-for-microservices-architecture-tutorial) to read more about how to register a new service.

## References:

  [How to write go code](https://golang.org/doc/code.html)  
  [Building Go for z System](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go)  
  [Consul](https://github.com/hashicorp/consul)  
  [Godeps](https://github.com/tools/godep)  
  [gox](https://github.com/mitchellh/gox)  	 
  https://www.consul.io/  	 
  https://github.com/hashicorp/consul