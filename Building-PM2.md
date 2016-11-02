# Building pm2 v2.0.18


The instructions provided below specify the steps to build pm2 v2.0.18 on Linux on the IBM z Systems for RHEL 7.1/7.2.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._


#### Section 1: Install the Dependencies

* RHEL 7.1/7.2

            sudo yum install -y wget

* Install Nodejs 

	Download the IBM Node.js SDK for Linux on System z 64-bit binary from [here](https://developer.ibm.com/node/sdk/#v12) and use the command below to install Node.js
		
			chmod +x ibm-1.2.0.15-node-v0.12.16-linux-s390x.bin
			./ibm-1.2.0.15-node-v0.12.16-linux-s390x.bin
	
	_**Note:** Nodejs requires java ibm jdk to be installed_ 

	Follow screen instruction to install `Node.js` to a folder (for example, `IBM_NODE_HOME`) and set PATH
	
			export PATH=$PATH:<IBM_NODE_HOME>/ibm/node/bin

		
#### Section 2: Installing pm2
            
1. Install pm2

        npm install pm2@2.0.18 -g     

2. Verification 

    1. Create a file `app.js`
	    ```
        var http = require('http');
        var server = http.createServer(function (request, response) {
          response.writeHead(200, {"Content-Type": "text/plain"});
          response.end("Welcome to pm2\n");
        });
        server.listen(80);
         console.log("Server running at http://127.0.0.1:80/");
       ``` 
       
    2. Run the application 
      
		    pm2 start app.js
      
		_**Note:** The application server will be started and serving at `http://127.0.0.1:80/`_
	3. Monitor the application
      
			  pm2 monit
      
### References:
 http://pm2.keymetrics.io/			
			
   