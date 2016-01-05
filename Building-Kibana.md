[Kibana](https://www.elastic.co/downloads/kibana) is an open source([Apache Licensed](https://github.com/elastic/kibana/blob/master/LICENSE.md)) analytics and visualization platform designed to work with Elasticsearch. It can easily perform advanced data analysis and visualize the data in a variety of charts, tables, and maps by searching, viewing, and interacting with data stored in Elasticsearch indices.

This recipe is for building Kibana (4.3.1) for Linux on z Systems (SLES12/SLES11/RHEL6/RHEL7)

### Dependencies:
   - IBM SDK for Node.js (0.12.7)
   - Elasticsearch (2.1.x or later)

### Build from Release


1. If IBM SDK for Node.js is already installed, go to step 3; otherwise go to http://www.ibm.com/developerworks/web/nodesdk/, click `Download now` for `Linux on z Systems` and `Download Version 1.2`, follow the instruction to download IBM SDK for Node.js 64-bit version and move to Linux system.

2. On Linux, install `Node.js`

        $ chmod +x ibm-1.2.0.5-node-v0.12.7-linux-s390x.bin
        $ ./ibm-1.2.0.5-node-v0.12.7-linux-s390x.bin

     Follow screen instruction to install `Node.js` to a folder (for example <IBM_NODE_HOME>).

3. Get Kibana release package and extract

        $ wget https://download.elastic.co/kibana/kibana/kibana-4.3.1-linux-x64.tar.gz
        $ tar xvf kibana-4.3.1-linux-x64.tar.gz

    If  `wget` is not installed, perform

        $ yum install wget              # for RHEL or
        $ zypper install wget           # for SLES

4. Replace `Node.js` in the package with the installed IBM SDK for Node.js.

        $ cd kibana-4.3.1-linux-x64
        $ mv node node_old         # rename the node
        $ ln -s <IBM_NODE_HOME>/ibm/node  node

   This creates a new folder `node` that links to installed IBM `Node.js`


5. Now Kibana is ready to execute, make sure an Elasticsearch instance is running, and update the Kibana configuration file `config/kibana.yml`  to set `elasticsearch_url` to the Elasticsearch host.

        $ bin/kibana     # start Kibana
    Open your browser at http://yourhost.com:5601 to make sure Kibana works.


