The [MongoDB Ruby Driver](http://docs.mongodb.org/ecosystem/drivers/ruby/) can be built for Linux on z Systems running SLES 12, SLES 11, RHEL 7 and RHEL 6 by following these instructions.  Versions 1.12 and 2.1.0 have been successfully built & tested this way. 

_**General Notes:**_ 	
_i) When following the steps below please use a standard permission user unless otherwise specified._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_  
_iii) For convenience `vi` has been used in the instructions below when editing files, replace with your desired editing program if required_


1. Install the build time dependencies 

    On RHEL 7.1 & 6.6 systems:
    ```shell
    sudo yum install ruby ruby-devel make gcc gcc-c++
    ```
    On SLES 12 & 11 systems:
    ```shell
    sudo zypper install ruby ruby-devel make gcc gcc-c++
    ```
  You may already have some of these packages installed - just install any that are missing.

2. Build Ruby from source (**only for Mongo 2.x and RHEL 6.6 / SLES 11**)

    Ruby MongoDB Driver 2.x requires Ruby version >= 1.9.3 and the RPM packages available for RHEL 6.6 and SLES 11 are too low level. Instructions for building Ruby 2.2.1 are available [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby).
3. Correct the gem environment for a standard user (skip if on RHEL 7.1)

    ```shell
    export GEM_HOME=/home/<USER>/.gem/ruby
    export PATH=/home/<USER>/.gem/ruby/bin:$PATH
    ```
    _Where `<USER>` is the standard user you are installing under._  
    _**Note:** Run `gem env` to verify the state of the environment, if later on you have issues installing / running ruby gems please ensure the environment is set correctly._
4. Install the mongo gem (and dependant bson gem)

    For the `1.12` version install just the mongo gem below:
    ```shell
    gem install mongo -v '1.12'
    ```
    And for the `2.1.0` version of the mongo gem you need to specify an additional gem (this is because the bson gem appears to change relatively quickly and so the instructions to disable native support can get outdated):
    ```shell
    gem install mongo -v '2.1.0'
    ```
    _**Note:** If the `-v` flag is omitted entirely the latest version will be installed

### Basic validation test
    
The example code section given below is used to perform a basic test to ensure that the MongoDB Ruby Driver is working as expected, and can connect to, modify and query a MongoDB server.

1. ***Prerequisites***

    The MongoDB Driver obviously needs access to a running MongoDB Server, either on your local server or a remote system. The following commands are an example of how to start up a MongodDB server and then connect to it with the client shell, but note that MongoDB has not been installed as part of these instructions, and typically you would be running MongoDB on a remote server.

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
    
    Create a file named test.rb with the content shown below.  This code connects to a MongoDB server, inserts some documents and then queries the database to read them back and display them. 
	
	_**Remember**_, if you are connecting to a remote server then you need to substitute the **`localhost`** with the hostname or IP address of the MongoDB server.

   As the Mongo Ruby Driver changed syntax between versions `1.x` a`2.x` the validation code has to be different.  
   
   **Mongo Ruby Driver 1.12**  
   
   ```shell
    require 'mongo'
    include Mongo

    server="localhost"
    test_db='ibm_test_db'
    collection='mongodb_ruby_driver_1_x'

    db = MongoClient.new(server, "27017").db(test_db)

    db.drop_collection(collection)

    coll = db.create_collection(collection, :capped => true, :size => 1024, :max => 12)

    header = {"company" => "IBM", "project" => "MongoDB Driver", "language" => "ruby", "version" => "1.12"}
    coll.insert(header)

    3.times { |i| coll.insert('line' => i+1) }

    coll.find().each { |line| p line }
    ```
    Execute the test program by (note; GEM_HOME may already have been set from following instructions above):
    ```shell
    export GEM_HOME=~/.gem/ruby
    ruby test.rb
    ```

    Executing the test program should produce output similar to this (the Object Ids will vary, but typically will be consecutive):
    ```shell
          ** Notice: The native BSON extension was not loaded. **

      For optimal performance, use of the BSON extension is recommended.

      To enable the extension make sure ENV['BSON_EXT_DISABLED'] is not set
      and run the following command:

        gem install bson_ext

      If you continue to receive this message after installing, make sure that
      the bson_ext gem is in your load path.
    {"_id"=>BSON::ObjectId('5603b52a2d24070006000001'), "company"=>"IBM", "project"=>"MongoDB Driver", "language"=>"ruby", "version"=>"1.12"}
    {"_id"=>BSON::ObjectId('5603b52a2d24070006000002'), "line"=>1}
    {"_id"=>BSON::ObjectId('5603b52a2d24070006000003'), "line"=>2}
    {"_id"=>BSON::ObjectId('5603b52a2d24070006000004'), "line"=>3}
    ```
    _**Note:** `Notice: The native BSON extension was not loaded` is expected as the `bson_ext` native port does not support Linux on z Systems_ for this version of the driver.
    
   **Mongo Ruby Driver 2.1.0**  
   
   ```shell
    require 'mongo'

    server="localhost"
    test_db='ibm_test_db'
    collection='mongodb_ruby_driver_2_x'

    server_addr=server + ":27017"

    Mongo::Logger.logger.level = ::Logger::FATAL   # hide DEBUG logging

    db = Mongo::Client.new([ server_addr ], :database => test_db)

    db[:collection].drop

    result = db[:collection].insert_one({ company: 'IBM' , project: 'MongoDB Driver', language: 'Ruby', version: '2.0.6'})

    3.times { |i| db[:collection].insert_one({ line: i+1 }) }

    db[:collection].find().each do |document|
       printf("%s\n", document) #=> Yields a BSON::Document.
    end
    ```
    
    Execute the test program by:
    ```shell
    export GEM_HOME=~/.gem/ruby
    ruby test.rb
    ```
    Executing the test program should produce output similar to this (the Object Ids will vary, but typically will be consecutive):
    ```shell
    D, [2015-09-24T08:49:10.487851 #26] DEBUG -- : MONGODB | Adding szptct05:27017 to the cluster.
D, [2015-09-24T08:49:10.493738 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.drop | STARTED | {"drop"=>"collection"}
D, [2015-09-24T08:49:10.495240 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.drop | SUCCEEDED | 0.001415781s
D, [2015-09-24T08:49:10.495636 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.insert | STARTED | {"insert"=>"collection", "documents"=>[{:company=>"IBM", :project=>"MongoDB Driver", :language=>"Ruby", :version=>"2.0.6", :_id=>BSON::ObjectId('5603b90602f44f001a000000')}], "ordered"=>true}
D, [2015-09-24T08:49:10.497339 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.insert | SUCCEEDED | 0.001638781s
D, [2015-09-24T08:49:10.497669 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.insert | STARTED | {"insert"=>"collection", "documents"=>[{:line=>1, :_id=>BSON::ObjectId('5603b90602f44f001a000001')}], "ordered"=>true}
D, [2015-09-24T08:49:10.498009 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.insert | SUCCEEDED | 0.000285313s
D, [2015-09-24T08:49:10.498320 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.insert | STARTED | {"insert"=>"collection", "documents"=>[{:line=>2, :_id=>BSON::ObjectId('5603b90602f44f001a000002')}], "ordered"=>true}
D, [2015-09-24T08:49:10.500990 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.insert | SUCCEEDED | 0.002610438s
D, [2015-09-24T08:49:10.501400 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.insert | STARTED | {"insert"=>"collection", "documents"=>[{:line=>3, :_id=>BSON::ObjectId('5603b90602f44f001a000003')}], "ordered"=>true}
D, [2015-09-24T08:49:10.501748 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.insert | SUCCEEDED | 0.000301532s
D, [2015-09-24T08:49:10.502139 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.find | STARTED | {"find"=>"collection", "filter"=>{}}
D, [2015-09-24T08:49:10.502586 #26] DEBUG -- : MONGODB | szptct05:27017 | ibm_test_db.find | SUCCEEDED | 0.00036587699999999997s
{"company"=>"IBM", "project"=>"MongoDB Driver", "language"=>"Ruby", "version"=>"2.0.6", "_id"=>BSON::ObjectId('5603b90602f44f001a000000')}
{"line"=>1, "_id"=>BSON::ObjectId('5603b90602f44f001a000001')}
{"line"=>2, "_id"=>BSON::ObjectId('5603b90602f44f001a000002')}
{"line"=>3, "_id"=>BSON::ObjectId('5603b90602f44f001a000003')}
    ```
    _**Note:** The DEBUG messages are expected from the driver, and unlike `BSON is using the pure Ruby implementation` is expected due to our disabling native earlier_