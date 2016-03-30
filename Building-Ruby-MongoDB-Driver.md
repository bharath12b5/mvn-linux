The [MongoDB Ruby Driver](http://docs.mongodb.org/ecosystem/drivers/ruby/) can be built for Linux on z Systems running SLES 12, SLES 11, RHEL 7 and RHEL 6 by following these instructions.  Versions 2.2.4 have been successfully built & tested this way. 

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

    Ruby MongoDB Driver 2.x requires Ruby version >= 1.9.3 and the RPM packages available for RHEL 6.6 and SLES 11 are too low level. Instructions for building Ruby 2.3.0 are available [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby).
3. Correct the gem environment for a standard user (skip if on RHEL 7.1)

    ```shell
    export GEM_HOME=/home/<USER>/.gem/ruby
    export PATH=/home/<USER>/.gem/ruby/bin:$PATH
    ```
    _Where `<USER>` is the standard user you are installing under._  
    _**Note:** Run `gem env` to verify the state of the environment, if later on you have issues installing / running ruby gems please ensure the environment is set correctly._
4. Install the mongo gem (and dependent bson gem)

    For the `2.2.4` version install mongo gem as given below:
    ```shell
    gem install mongo -v '2.2.4'
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

   **Mongo Ruby Driver 2.2.4**  
   
   ```shell
    require 'mongo'

    server="localhost"
    test_db='ibm_test_db'
    collection='mongodb_ruby_driver_2_x'

    server_addr=server + ":27017"

    Mongo::Logger.logger.level = ::Logger::FATAL   # hide DEBUG logging

    db = Mongo::Client.new([ server_addr ], :database => test_db)

    db[:collection].drop

    result = db[:collection].insert_one({ company: 'IBM' , project: 'MongoDB Driver', language: 'Ruby', version: '2.2.4'})

    3.times { |i| db[:collection].insert_one({ _id: i+1, line: i+1 }) }

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
    {"_id"=>BSON::ObjectId('000002aafe4122006d000002'), "company"=>"IBM", "project"=>"MongoDB Driver", "language"=>"Ruby", "version"=>"2.2.4"}
    {"_id"=>1, "line"=>1}
    {"_id"=>2, "line"=>2}
    {"_id"=>3, "line"=>3}

    ```
