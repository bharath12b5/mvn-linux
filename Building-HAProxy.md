**General Notes**: These instructions have been tested for HAProxy version 1.5.14 on RHEL 6.6/7 and SLES 11.3/12. When following the steps below please use a standard permission user unless otherwise specified.

## Downloading, Building and Installing HAProxy 1.5.14

1. Download and unpack the HAProxy 1.5.14 source package.
    ```
    wget http://www.haproxy.org/download/1.5/src/haproxy-1.5.14.tar.gz
    tar xzvf haproxy-1.5.14.tar.gz
    cd haproxy-1.5.14/
    ```

   Alternatively, if the latest development code is preferred:
    ```
    git clone http://git.haproxy.org/git/haproxy.git/
    cd haproxy/
    ```

2. Build and install HAProxy
    ```
    make TARGET=linux26
    sudo make install
    ```

3. **(Optional)** HAProxy will be installed in /usr/local/sbin; depending upon user preferences and conventions, it may be necessary to either update PATH or create links to the executable files.


## Simple Round-Robin Load Balancing Test
*Adapted from [Servers for Hackers](https://serversforhackers.com/load-balancing-with-haproxy)*

1. Prepare a simple set of servers serving the same webpage.

   For example, this simple node.js script will serve a small text page, including the serving address and port, on the machine's ports 9000-9002:
    ```
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
    ```

2. Create a file ```haproxy.conf``` to serve as the configuration file for the test:

   This is a simple round-robin configuration using three servers:
    ```
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
    ```

   This assumes that the node.js script from step 1 is being run on the same machine, serving three almost-identical webpages on ports 9000-9002. If this is not the case, replace the declarations for s1-s3 accordingly.

3. Run HAProxy with the provided configuration.
    ```
    haproxy -f haproxy.config
    ```

   Note that this will normally need to be done as root, or HAProxy will not have authority to access one or more ports, such as 80 and 8080.

4. Navigate a web browser to the address of the server running HAProxy. The browser display the webpage from the first of the servers in ```haproxy.config```. If the browser is refreshed, it should cycle through the provided servers in a round-robin fashion. If the node.js script from step 1 was used, the serving port number should change visibly with these refreshes.

## References:
* [HAProxy.org](http://www.haproxy.org/)
* [Servers for Hackers - Load Balancing with HAProxy](https://serversforhackers.com/load-balancing-with-haproxy)