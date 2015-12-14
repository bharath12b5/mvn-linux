
The PHP driver for MongoDB can be built for Linux on z Systems running RHEL 7, RHEL 6, SLES12 or SLES 11 by following these instructions.  Version 1.6.11 has been successfully built & tested this way.
More information on the PHP driver is available at http://docs.mongodb.org/ecosystem/drivers/php/ and the source code can be obtained from http://github.com/mongodb/mongo-php-driver

_**General Notes:**_ 	

i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it

## Building the PHP Driver for MongoDB

### Obtain pre-built dependencies and create working directory

1. Use the following commands to obtain required pre-built dependencies

    Note that if you intend to "Build with Cyrus SASL (MongoDB Enterprise Authentication) support" then you will need to add cyrus-sasl-devel to the list of dependencies shown below.

    For RHEL 7.1 or RHEL 6.6

    ```shell
    sudo yum install gcc make openssl-devel php-devel php-pear tar wget
    ```
	
      For SLES 12

    ```shell
    sudo zypper install gcc make php5-devel php5-pear tar wget
    ```
	
    For SLES 11

    ```shell
    sudo zypper install gcc make php53-devel php53-pear tar wget
    ```
	
2. Create temporary installation directory

    ```shell
    mkdir /<source_root>
    ```

### Product Build - PHP Driver for MongoDB

The current version of the PHP driver source code (1.6.11) has a few issues waiting to be resolved (PHP-1483, PHP-1484, PHP-1485).
A suggested temporary work around for these outstanding issues is to build from source code, having applied patches to fix/work around the issues.

Instructions are given to EITHER download source code to patch and build, OR to download current version from the PHP Extension Community Library (PECL). This method is easier and faster once it is fixed.

#### EITHER: Download, patch and build source code for PHP driver

1. Download the PHP driver source code

    ```shell
    cd /<source_root>/
    wget -O 1.6.11.tar.gz https://github.com/mongodb/mongo-php-driver-legacy/archive/1.6.11.tar.gz
    tar -xf 1.6.11.tar.gz
    mv mongo-php-driver-legacy-1.6.11 mongo-php-driver
    ```

1. Configure the PHP driver

    Note that if you wish to "Build with Cyrus SASL (MongoDB Enterprise Authentication) support" then you will need to append `--with-mongo-sasl=/usr/local` to the `configure` command shown below.

    ```shell
    cd mongo-php-driver
    phpize
    ./configure
    ```

1. Patch the PHP driver source code

    Work around for the outstanding issues (mentioned above) by amending the source code by issuing the following commands:-

    ```shell
    cd /<source_root>/mongo-php-driver
    sed -i "s/int chunk_size;/long chunk_size;/" php_mongo.h
    sed -i "s/MonGlo(chunk_size)/(int)(MonGlo(chunk_size))/" gridfs/gridfs.c
    sed -i "s/define MONGO_32(b) (b)/define MONGO_32(b) ((b) \& 0xffffffff)/" bson.h
    sed -i "s/define MONGO_32(b) (b)/define MONGO_32(b) ((b) \& 0xffffffff)/" mcon/bson_helpers.h
    sed -i "s/memcpy(data + 7, P + 2, 2);/memcpy(data + 7, P + 3, 1);\n\tmemcpy(data + 8, P + 2, 1);/" types/id.c
    ```

1. Build and install the PHP driver

    ```shell
    cd /<source_root>/mongo-php-driver
    make
    sudo make install
    ```

#### OR: Use current version downloaded from PECL

Follow these instructions to directly obtain the MongodDB PHP driver from PECL. 

_**Note:**_ At time of writing this is at version 1.6.11, which will exhibit known issues (mentioned above). In particular, it will :

- show the task id as a Big Endian number within the OID (debatable if this is incorrect behaviour, on a Big Endian Z machine, but currently fails unit tests, due to differing from behaviour on x86 architecture), and 
- GridFS file services will not work correctly, producing an error similar to:

   ```shell
    PHP Fatal error:  Allowed memory size of 134217728 bytes exhausted (tried to allocate 74 bytes) in
   ```

1. Download and build PHP driver from PECL

    First ensure that pecl.php.net is up to date by running `pecl channel-update` and then use `pecl` to build and install the php mongo extension, as follows:

    ```shell
    sudo pecl channel-update pecl.php.net
    sudo pecl install mongo
    ```

    Note that you may be prompted to choose whether to "Build with Cyrus SASL (MongoDB Enterprise Authentication) support?", in which case you should respond with "yes" or "no" according to your needs.


#### Enable PHP extension

In both cases, whether downloaded and installed from PECL, or built from source code, the PHP Driver extension needs to be enabled by adding a line to php.ini

For RHEL

   ```shell
      echo -e "\n; PHP driver for MongoDB\nextension=mongo.so" | sudo tee -a /etc/php.ini
   ```
	
For SLES

   ```shell
      echo -e "\n; PHP driver for MongoDB\nextension=mongo.so" | sudo tee -a /etc/php5/cli/php.ini
   ```
	

### Basic validation test
    
The example code section given below is used to perform a basic test to ensure that the MongoDB C Driver is working as expected, and can connect to, modify and query a MongoDB server.

1. ***Prerequisites***

    The MongoDB Driver obviously needs access to a running MongoDB server, either on your local server or a remote system. The following commands are an example of how to start up a MongodDB server and then connect to it with the client shell, but note that MongoDB has not been installed as part of these instructions, and typically you would be running MongoDB on a remote server.

    ```shell
    mongod > /tmp/mongodb.log &
    mongo --host localhost 
    ```
    Which would typically give a command prompt such as:
    
    ```shell
    MongoDB shell version: 2.4.10 connecting to: localhost:27017/test > 
    ```
    The example code below will need to be modified to use your remote server hostname or IP address instead of "localhost", if you are attempting to connect to your own (remote) server.
    
2. ***The Test Code***
    
    Create a file named 'test.php' in `/<source_root>` with the content shown below. 

_**Remember**_, if you are connecting to a remote server then you need to substitute the `localhost` with the hostname or IP address of the MongoDB server.

_**In addition**_, create a short text file in the same directory, named `example.txt`

   ```shell
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

    $col->insert(array('company' => 'IBM', 'project' => 'MongoDB Driver', 'language' => 'PHP', 'version' => '1.6.11'));

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
   ```

3. ***Execute*** 

    Execute the test script by:

    ```shell
    cd /<source_root>/
    php test.php
    ```

    Executing the script should produce output similar to this (the Object Ids will vary, but typically will be consecutive):

    ```shell
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
      string(6) "1.6.11"
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
    ```

### [Optional] Clean up

Once you have installed the PHP driver outside `/<source_root>/` then `/<source_root>/` may be deleted.


###References:

[PHP_MongoDB_Driver-website](http://docs.mongodb.org/ecosystem/drivers/php/)

[PHP_MongoDB_Driver-source](http://github.com/mongodb/mongo-php-driver)

[PHP Manual](http://php.net/manual/en/index.php)