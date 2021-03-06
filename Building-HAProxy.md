<!---PACKAGE:HAProxy--->
<!---DISTRO:SLES 12:1.7--->
<!---DISTRO:SLES 11:1.7--->
<!---DISTRO:RHEL 7.1:1.7--->
<!---DISTRO:RHEL 6.6:1.7--->
<!---DISTRO:Ubuntu 16.x:Distro, 1.7--->

# Building HAProxy

Below versions of HAProxy are available in respective distributions at the time of this recipe creation:

* Ubuntu 16.04 has `1.6.3`
* Ubuntu 16.10 has `1.6.8`

The instructions provided below specify the steps to build HAProxy 1.7.1 on the IBM z Systems for RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP4 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	 
 * When following the steps below please use a standard permission user unless otherwise specified._

 * A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##Step 1: Building and Installing HAProxy
####1.1) Install the build dependencies

* RHEL 6.8 and RHEL 7.1/7.2/7.3
  ```
  sudo yum install  wget tar make  gcc 
  ```

* SLES 11-SP4 and SLES 12/12-SP1/12-SP2
  ```
  sudo zypper install  wget tar make  gcc
  ```

* Ubuntu 16.04/16.10
  ```
  sudo apt-get update
  sudo apt-get install  wget tar make  gcc
  ```
  
####1.2) Download and unpack the required HAProxy source package
  ```
  cd /<source_root>/
  wget http://www.haproxy.org/download/1.7/src/haproxy-1.7.1.tar.gz
  tar xzvf haproxy-1.7.1.tar.gz
  cd haproxy-1.7.1/
  ```
####1.3) Build and install HAProxy
 
  ```
  make TARGET=linux26
  sudo make install
  ```

  **Note:** HAProxy will be installed in `/usr/local/sbin`. Depending upon user preferences and conventions, it may be necessary to either update PATH or create links to the executable files.


## Step 2: Testing: Simple Round-Robin Load Balancing Test

_Adapted from [Servers for Hackers](https://serversforhackers.com/load-balancing-with-haproxy)_

####2.1)Prepare a simple set of servers serving the same webpage

    For example, this simple node.js script will serve a small text page, including the serving address and port, on the machine's ports 9000-9002:

        var http = require('http');

        function serve(ip, port)
        {
            http.createServer(function (req, res)
            {
                res.writeHead(200, {'Content-Type': 'text/plain'});
                res.write(JSON.stringify(req.headers));
                res.end("\nThere's no place like "+ip+":"+port+"\n");
            }).listen(port, ip);
            console.log('Server running at http://'+ip+':'+port+'/');
        }

        // Create three servers for
        // the load balancer, listening on any
        // network on the following three ports
        serve('0.0.0.0', 9000);
        serve('0.0.0.0', 9001);
        serve('0.0.0.0', 9002);

    **Note:** *To install Node.js, click [here](http://developer.ibm.com/node/sdk/)*
####2.2)Create a file `haproxy.config` to serve as the configuration file for the test in the  `/<source_root>/` directory

    This is a simple round-robin configuration using three servers:

        global
            daemon
            maxconn 256

        defaults
            mode http
            timeout connect 5000ms
            timeout client 50000ms
            timeout server 50000ms

        frontend http-in
            bind *:80
            default_backend neo4j

        backend neo4j
            mode http
            balance roundrobin
            option forwardfor
            http-request set-header X-Forwarded-Port %[dst_port]
            option httpchk HEAD / HTTP/1.1\r\nHost:localhost
            server s1 127.0.0.1:9000 maxconn 32
            server s2 127.0.0.1:9001 maxconn 32
            server s3 127.0.0.1:9002 maxconn 32

        listen admin
            bind *:8080
            stats enable

    **Note:** *This assumes that the node.js script from step 2.1 is being run on the same machine, serving three almost-identical webpages on ports 9000-9002\. If this is not the case, replace the declarations for s1-s3 accordingly.*

####2.3) Run HAProxy with the provided configuration
      
        cd /<source_root>/
        haproxy -f haproxy.config

  **Note:** *This will normally need to be done as root, or HAProxy will not have authority to access one or more ports, such as 80 and 8080.*

####2.4) Navigate a web browser to the address of the server running HAProxy 

The browser displays the webpage from the list of the servers in `haproxy.config`. If the browser is refreshed, it should cycle through the provided servers in a round-robin fashion. If the node.js script from step 2.1 was used, the serving port number should change visibly with these refreshes.

## References:

*   [HAProxy.org](http://www.haproxy.org/)
*   [Servers for Hackers - Load Balancing with HAProxy](https://serversforhackers.com/load-balancing-with-haproxy)
