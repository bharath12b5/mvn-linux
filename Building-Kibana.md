<!---PACKAGE:Kibana--->
<!---DISTRO:RHEL 6.6:4.5.1--->
<!---DISTRO:RHEL 7.1:4.5.1--->
<!---DISTRO:SLES 11:4.5.1--->
<!---DISTRO:SLES 12:4.5.1--->

[Kibana](https://www.elastic.co/downloads/kibana) is an open source([Apache Licensed](https://github.com/elastic/kibana/blob/master/LICENSE.md)) analytics and visualization platform designed to work with Elasticsearch. It can easily perform advanced data analysis and visualize the data in a variety of charts, tables, and maps by searching, viewing, and interacting with data stored in Elasticsearch indices.

The instructions provided below specify the steps to build Kibana (4.5.1) on Linux on the IBM z Systems for RHEL 6/7.2, SLES 11/12 and Ubuntu 16.04.

### Dependencies:
   - IBM SDK for Node.js (0.12.7 or later)
   - Elasticsearch (2.3.x or later)
   - Java


### Building Kibana

1. Use the following commands to obtain dependencies

    For RHEL 6.6 and RHEL 7.2
    ```shell
    sudo yum install -y wget tar java-1.7.1-ibm-devel
    ```
    For SLES 11
    ```shell
    sudo zypper install -y wget tar java-1.7.0-ibm-devel
    ```
    For SLES 12
    ```shell
    sudo zypper install -y wget tar java-1.7.1-ibm-devel
    ```
    For Ubuntu 16.04
    ```shell
	sudo apt-get update
    sudo apt-get install -y wget tar openjdk-8-jdk
    ```

2. If IBM SDK for Node.js is already installed, go to step 4; otherwise go to http://www.ibm.com/developerworks/web/nodesdk/, click `Download now` for `Linux on z Systems` and `Download Version 1.2`, follow the instruction to download IBM SDK for Node.js 64-bit version and move to Linux system.

3. On Linux, install `Node.js`

        $ chmod +x ibm-1.2.0.10-node-v0.12.12-linux-s390x.bin
        $ ./ibm-1.2.0.10-node-v0.12.12-linux-s390x.bin

     Follow screen instruction to install `Node.js` to a folder (for example, `IBM_NODE_HOME`).

4. Get Kibana release package and extract

        $ wget https://download.elastic.co/kibana/kibana/kibana-4.5.1-linux-x64.tar.gz
        $ tar xvf kibana-4.5.1-linux-x64.tar.gz
  
5. Replace `Node.js` in the package with the installed IBM SDK for Node.js.

        $ cd kibana-4.5.1-linux-x64
        $ mv node node_old         # rename the node
        $ ln -s <IBM_NODE_HOME>/ibm/node  node

   This creates a new folder `node` that links to installed IBM `Node.js`


5. Now Kibana is ready to execute, make sure an Elasticsearch instance is running, and update the Kibana configuration file `config/kibana.yml`  to set `elasticsearch.url` to the Elasticsearch host.

        $ bin/kibana     # start Kibana
    Open your browser and go to http://yourhost.com:5601 to make sure Kibana works.


