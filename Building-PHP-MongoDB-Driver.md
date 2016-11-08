##  Building the PHP Driver for MongoDB

The instructions provided below specify the steps to build PHP driver for MongoDB version 1.6.12 on Linux on the IBM z Systems for RHEL 7.1, RHEL 7.2, RHEL 6.7, SLES 12, SLES 12-SP1, SLES 11-SP3 and Ubuntu 16.04.

More information on the PHP driver is available at http://docs.mongodb.org/ecosystem/drivers/php/ and the source code can be obtained from http://github.com/mongodb/mongo-php-driver

**_General Notes_:**

i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary write-able directory anywhere you'd like to place it._


***
### Obtain pre-built dependencies and create working directory
1. Use the following commands to obtain required pre-built dependencies

     Note that if you intend to "Build with Cyrus SASL (MongoDB Enterprise Authentication) support" then you will need to add cyrus-sasl-devel to the list of dependencies shown below.


  For RHEL 7.1/7.2 & RHEL 6.7

		sudo yum install gcc make openssl-devel php-devel php-pear tar wget git
		              
  For SLES 12 & SLES 12-SP1

		sudo zypper install gcc make php5-devel php5-pear tar wget git
		
  For SLES 11-SP3

		sudo zypper install gcc make php53-devel php53-pear tar wget git
		
  For Ubuntu 16.04
  
		sudo apt-get update
		sudo apt-get install gcc make tar wget git autoconf libxml2 libxml2-dev

2. Create temporary installation directory

		mkdir /<source_root>
		
3. Build and install php5.6.8 **(Only for Ubuntu 16.04)** 

		cd /<source_root>
		wget http://www.php.net/distributions/php-5.6.8.tar.gz
		tar xvzf php-5.6.8.tar.gz
		cd /<source_root>/php-5.6.8 
		./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php
		make
		sudo make install
		sudo cp /<source_root>/php-5.6.8/php.ini-development /usr/local/php/php.ini
		export PATH=/usr/local/php/bin:$PATH

		   
### Product Build - PHP Driver for MongoDB

The current version of the PHP driver source code (1.6.12) has a few issues waiting to be resolved (PHP-1483, PHP-1484, PHP-1485). A suggested temporary work around for these outstanding issues is to build from source code, having applied patches to fix/work around the issues.

Instructions are given to EITHER download source code to patch and build, OR to download current version from the PHP Extension Community Library (PECL). This method is easier and faster once it is fixed.

#### EITHER: Download, patch and build source code for PHP driver
1. Download the PHP driver source code

        

        cd /<source_root>/
        git clone https://github.com/mongodb/mongo-php-driver-legacy.git
        cd mongo-php-driver-legacy
        git checkout 1.6.12



2. Configure the PHP driver

    Note that if you wish to "Build with Cyrus SASL (MongoDB Enterprise Authentication) support" then you will need to append `--with-mongo-sasl=/usr/local` to the `configure` command shown below.
    
	    cd mongo-php-driver-legacy
	    phpize
	    ./configure

3. Patch the PHP driver source code     
        Work around for the outstanding issues (mentioned above) by amending the source code by issuing the following commands:-                        

	    sed -i "s/int chunk_size;/long chunk_size;/" php_mongo.h
	    sed -i "s/MonGlo(chunk_size)/(int)(MonGlo(chunk_size))/" gridfs/gridfs.c
	    sed -i "s/define MONGO_32(b) (b)/define MONGO_32(b) ((b) \& 0xffffffff)/" bson.h
	    sed -i "s/define MONGO_32(b) (b)/define MONGO_32(b) ((b) \& 0xffffffff)/" mcon/bson_helpers.h
	    sed -i "s/memcpy(data + 7, P + 2, 2);/memcpy(data + 7, P + 3, 1);\n\tmemcpy(data + 8, P + 2, 1);/" types/id.c
	    
	 
	    
4. Build and install the PHP driver

        cd /<source_root>/mongo-php-driver-legacy
        make 
        sudo make install
       

	 
### OR: Use current version downloaded from PECL

Follow these instructions to directly obtain the MongodDB PHP driver from PECL.


_**Note:**_ At time of writing this is at version 1.6.12, which will exhibit known issues (mentioned above). In particular, it will :
* show the task id as a Big Endian number within the OID (debatable if this is incorrect behaviour, on a Big Endian Z machine, but currently fails unit tests, due to differing from behaviour on x86 architecture), and 
* GridFS file services will not work correctly, producing an error similar to:

	       PHP Fatal error:  Allowed memory size of 134217728 bytes exhausted (tried to allocate 74 bytes) in


1. Download and build PHP driver from PECL

First ensure that pecl.php.net is up to date by running pecl channel-update and then use pecl to build and install the php mongo extension, as follows:

	       sudo pecl channel-update pecl.php.net
	       sudo pcel install mongo


Note that you may be prompted to choose whether to "Build with Cyrus SASL (MongoDB Enterprise Authentication) support?", in which case you should respond with "yes" or "no" according to your needs.
		   
### Enable PHP extension

In both cases, whether downloaded and installed from PECL, or built from source code, the PHP Driver extension needs to be enabled by adding a line to php.ini

For RHEL
	       
	        echo -e "\n; PHP driver for MongoDB\nextension=mongo.so" | sudo tee -a /etc/php.ini


For SLES
	       
	        echo -e "\n; PHP driver for MongoDB\nextension=mongo.so" | sudo tee -a /etc/php5/cli/php.ini
			

For Ubuntu 16.04

            
			echo -e "\n; PHP driver for MongoDB\nextension=mongo.so" | sudo tee -a  /usr/local/php/php.ini


### Basic validation test	  

The example code section given below is used to perform a basic test to ensure that the MongoDB C Driver is working as expected, and can connect to, modify and query a MongoDB server.


**1. Prerequisites**



The MongoDB Driver obviously needs access to a running MongoDB server, either on your local server or a remote system. The following commands are an example of how to start up a MongodDB server and then connect to it with the client shell, but note that MongoDB has not been installed as part of these instructions, and typically you would be running MongoDB on a remote server.




	       
	        mongod > /tmp/mongodb.log &
	        mongo --host localhost 

Which would typically give a command prompt such as:

    MongoDB shell version: 2.4.10 connecting to: localhost:27017/test >
 
 
 The example code below will need to be modified to use your remote server hostname or IP address instead of "localhost", if you are attempting to connect to your own (remote) server.


**2. The test code**


  Create a file named 'test.php' in `/<source_root>` with the content shown below. 
  
 **Remember**, if you are connecting to a remote server then you need to substitute the `localhost` with the hostname or IP address of the MongoDB server.
  
  **In addition,** create a short text file in the same directory, named `example.txt`



    <?php

    // Config
    $dbhost = 'localhost';
    $dbname = 'ibm_test_db';
    $collection = 'mongodb_php_client';
    $test_msg = 'test message';

    // Connect to test database
    $m = new MongoClient("mongodb://$dbhost",array("connect" => TRUE));
    $db = $m->$dbname;

    // Clear collection, if it exists, and create from new
    $db->selectCollection($collection)->drop();
    $col = $db->createCollection( $collection, array( 'capped' => true, 'size' => 10*1024, 'max' => 15));

    $col->insert(array('company' => 'IBM', 'project' => 'MongoDB Driver', 'language' => 'PHP', 'version' => '1.6.12'));

    for ($i = 0; $i < 3; $i++) {
        $col->insert(array('line' => $i));
    }

    $cursor = $col->find();

    foreach ($cursor as $doc) {
        var_dump ($doc);
    } 

    // Store example file in db
    $gridfs = $m->selectDB($dbname)->getGridFS();
    $id = $gridfs->storeFile('example.txt', array('contentType' => 'plain/text'));

    // Get file from db and print the contents of the file
    $gridfsFile = $gridfs->get($id);
    echo "\nFile contents:\n";
    echo $gridfsFile->getBytes();
    ?>




**3. Execute**

Execute the test script by :

    cd /<source_root>/
    php test.php


Executing the script should produce output similar to this (the Object Ids will vary, but typically will be consecutive):
    

    array(5) {
      ["_id"]=>
      object(MongoId)#7 (1) {
        ["$id"]=>
        string(24) "5600314dbd6a3b01008b4567"
      }
      ["company"]=>
      string(3) "IBM"
      ["project"]=>
      string(14) "MongoDB Driver"
      ["language"]=>
      string(3) "PHP"
      ["version"]=>
      string(6) "1.6.12"
    }
    array(2) {
      ["_id"]=>
      object(MongoId)#8 (1) {
        ["$id"]=>
        string(24) "5600314dbd6a3b01008b4568"
      }
      ["line"]=>
      int(0)
    }
    array(2) {
      ["_id"]=>
      object(MongoId)#7 (1) {
        ["$id"]=>
        string(24) "5600314dbd6a3b01008b4569"
      }
      ["line"]=>
      int(1)
    }
    array(2) {
      ["_id"]=>
      object(MongoId)#8 (1) {
        ["$id"]=>
        string(24) "5600314dbd6a3b01008b456a"
      }
      ["line"]=>
      int(2)
    }

    File contents:
    <contents of your file...>


### [Optional] Clean up
Once you have installed the PHP driver outside `/<source_root>/` then `/<source_root>/` may be deleted.


## References:
[ PHP_MongoDB_Driver-website](http://docs.mongodb.org/ecosystem/drivers/php/)

[PHP_MongoDB_Driver-source](http://github.com/mongodb/mongo-php-driver)

[PHP Manual](http://php.net/manual/en/index.php)
