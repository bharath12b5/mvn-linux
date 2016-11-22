# Building PM2

The instructions provided below specify the steps to install PM2 v2.0.19 on Linux on the IBM z Systems for RHEL 7.1/7.2, SLES 12/12-SP1, and Ubuntu 16.04/16.10.

_**General Notes:**_  
_When following the steps below please use a standard permission user unless otherwise specified._

**1. Install the Dependencies**

For RHEL 7.1/7.2

 ```
 sudo yum install -y wget
 ```

For SLES 12/12-SP1

 ```
 sudo zypper install -y wget
 ```

For Ubuntu 16.04/16.10
 
 ```
 sudo apt-get install nodejs npm
 sudo ln -s `which nodejs` /usr/local/bin/node
 ```
			

* Install Nodejs (For RHEL 7.1/7.2 and SLES 12/12-SP1 only )

	Download the IBM Node.js SDK for Linux on System z 64-bit binary from [here](https://developer.ibm.com/node/sdk/#v12) and use the command below to install Node.js
		
			chmod +x ibm-1.2.0.15-node-v0.12.16-linux-s390x.bin
			./ibm-1.2.0.15-node-v0.12.16-linux-s390x.bin
	
	_**Note:** Nodejs requires java ibm jdk to be installed_ 

	Follow screen instruction to install `Node.js` to a folder (for example, `IBM_NODE_HOME`) and set PATH
	
			export PATH=$PATH:<IBM_NODE_HOME>/ibm/node/bin
		
**2. Install PM2**  

* RHEL 7.1/7.2 and SLES 12/12-SP1
     ```
     npm install pm2@2.0.19 -g
     ```
       
* Ubuntu 16.04/16.10
     ```
     sudo npm install pm2@2.0.19 -g
     ```

  _**Note:** You can also install previous versions of PM2. Example(To install version 2.0.18): `sudo npm install pm2@2.0.18 -g`_

**3. Verification**

  * Create a file `app.js`
  ```
var http = require('http');
var server = http.createServer(function (request, response) {
response.writeHead(200, {"Content-Type": "text/plain"});
response.end("Welcome to pm2\n");
});
server.listen(8080);
console.log("Server running at http://127.0.0.1:8080/");
  ```           
       
  * Run the application 
  ```     
pm2 start app.js
  ```

_**Note:** The application server will be started and serving at `http://127.0.0.1:8080/`_
    
  * Monitor the application

  ```
pm2 monit
  ```
      
### References:
 http://pm2.keymetrics.io/