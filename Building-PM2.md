# Building pm2 v2.0.18


The instructions provided below specify the steps to build pm2 v2.0.18 on Linux on the IBM z Systems for RHEL 7.2.

_**General Notes:**_  
i) _When following the steps below please use a standard permission user unless otherwise specified._


#### Section 1: Install the Dependencies

* RHEL 7.2

            sudo yum install -y git make wget php-cli bzip2 tar

* Install Nodejs 

	Download the IBM Node.js SDK for Linux on System z 64-bit binary from [here](https://developer.ibm.com/node/sdk/#v12) and use the command below to install Node.js
		
			chmod +x ibm-1.2.0.15-node-v0.12.16-linux-s390x.bin
			./ibm-1.2.0.15-node-v0.12.16-linux-s390x.bin
	
	Follow screen instruction to install `Node.js` to a folder (for example, `IBM_NODE_HOME`) and set PATH
	
			export PATH=$PATH:<IBM_NODE_HOME>/ibm/node/bin

		
#### Section 2: Building and installing pm2
            
1. Install GNU parallel needed for testcases 

            cd $HOME
            wget http://ftp.gnu.org/gnu/parallel/parallel-20160922.tar.bz2
            tar jxf parallel-20160922.tar.bz2
            cd parallel-20160922
            ./configure
            make
            sudo make install     

2. Build and Install

           cd $HOME
           git clone https://github.com/Unitech/pm2.git
           cd $HOME/pm2
           git checkout v2.0.18
           npm install
        
3. Add mocha binary to the PATH

           export PATH=$HOME/pm2/node_modules/mocha/bin:$PATH

4. Running Testcases (Optional)

    1. Replace the text `python3` with `python` in the following files

            $HOME/pm2/test/fixtures/extra-lang/apps.json
            $HOME/pm2/test/bash/extra-lang.sh
            $HOME/pm2/test/fixtures/yaml-configuration/apps.yaml
            $HOME/pm2/test/fixtures/yaml-configuration/apps.yml

    2. Edit file `$HOME/pm2/test/programmatic_commands.txt` as follows
   
      from

		    ...   
		    mocha ./programmatic/send_data_process.mocha.js
		    mocha ./programmatic/return.mocha.js
		    mocha ./programmatic/json_validation.mocha.js
		    ...

      to

		    ...
		    mocha ./programmatic/send_data_process.mocha.js
		    mocha ./programmatic/json_validation.mocha.js
		    ...

   3. Edit file `$HOME/pm2/lib/templates/Dockerfiles/Dockerfile-nodejs.tpl` as follows to use s390x/ubuntu image

      from

            FROM mhart/alpine-node:latest

            RUN apk update && apk add git && rm -rf /var/cache/apk/*
            RUN npm install pm2@next -g
            RUN mkdir -p /var/app

            WORKDIR /var/app

            COPY ./package.json /var/app
            RUN npm install
        
      to
        
            FROM s390x/ubuntu:latest

            RUN apt-get update && apt-get install -y gcc wget git make nodejs npm && npm install pm2@next -g && ln -s "$(which nodejs)" /usr/bin/node
        
            RUN mkdir -p /var/app

            WORKDIR /var/app
 
            COPY ./package.json /var/app/
            RUN npm install
        
    5. Run the testcases
 
            cd $HOME/pm2
            bash test/pm2_programmatic_tests.sh
            bash test/pm2_behavior_tests.sh
            bash test/parallel_programmatic_tests.sh
        