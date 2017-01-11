<!---PACKAGE:PM2--->
<!---DISTRO:SLES 11.X:2.2.3--->
<!---DISTRO:SLES 12.X:2.2.3--->
<!---DISTRO:RHEL 6.X:2.2.3--->
<!---DISTRO:RHEL 7.X:2.2.3--->
<!---DISTRO:Ubuntu 16.X:2.2.3--->

# Building PM2

The instructions provided below specify the steps to install PM2 v2.2.3 on Linux on the IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._
* _At the time of creating this recipe latest version was `2.2.3`, you can also use `npm install pm2 -g` to install latest version of PM2._

##Step 1: Installing PM2
####1.1) Install Node.js

* RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4 and SLES 12/12-SP1/12-SP2

  Download the IBM Node.js SDK for Linux on System z 64-bit binary from [here](https://developer.ibm.com/node/sdk/) and use the command below to install Node.js
  ```bash
  chmod +x ibm-1.2.0.15-node-v0.12.16-linux-s390x.bin
  ./ibm-1.2.0.15-node-v0.12.16-linux-s390x.bin
  ```
  _**Note:**_
  * _Node.js requires [Java IBM SDK](http://www.ibm.com/developerworks/java/jdk/linux/download.html) to be installed._
  * _Node.js requires GCC 4.8 or newer. RHEL 6.8 and SLES 11-SP4 ship older versions of GCC, so it is necessary to build and install a newer GCC. Refer the "Building and Installing GCC" part of the [GCCGO](https://github.com/linux-on-ibm-z/docs/wiki/Building-gccgo) recipe._

  Follow screen instruction to install `Node.js` to a folder (for example, `IBM_NODE_HOME`) and set PATH
  ```bash
  export PATH=$PATH:<IBM_NODE_HOME>/ibm/node/bin
  ```

* Ubuntu 16.04/16.10
 
 ```bash
 sudo apt-get install nodejs npm
 sudo ln -s `which nodejs` /usr/local/bin/node
 ```

####1.2) Install PM2

* RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4 and SLES 12/12-SP1/12-SP2
  ```bash
  npm install pm2@2.2.3 -g
  ```

* Ubuntu 16.04/16.10
  ```bash
  sudo npm install pm2@2.2.3 -g
  ```

##Step 2: Verification(Optional)

####2.1) Create a file `app.js`

  ```js
  var http = require('http');
  var server = http.createServer(function (request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.end("Welcome to pm2\n");
  });
  server.listen(8080);
  console.log("Server running at http://127.0.0.1:8080/");
  ```           

####2.2) Run the application 

  ```bash     
  pm2 start app.js
  ```

  _**Note:** The application server will be started and serving at `http://127.0.0.1:8080/`._
    
####2.3) Monitor the application

  ```bash
  pm2 monit
  ```
      
### References:
 http://pm2.keymetrics.io/
