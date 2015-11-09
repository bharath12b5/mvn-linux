#Building MongoDB C++ Driver

The MongoDB C++ Driver can be built for Linux on z Systems running SLES12, RHEL7 and RHEL6 by following these instructions. The 'legacy' driver version legacy-1.0.5 has been successfully built & tested this way.


_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified._  


1. **Install build time dependencies**

   For **RHEL 7.1**
  ```shell
  sudo yum install automake boost-devel libtool gcc-c++ git make wget
  ```
   For **RHEL 6**
  ```shell
  sudo yum install automake boost-devel libtool gcc-c++ git make tar wget which
  ```
  For **SLES 12**
  ```shell
  sudo zypper install automake boost-devel libtool gcc-c++ git make scons
  ```
  For **SLES 11**
  ```shell
  Known not to work on SLES 11
  ```
 
  You may already have these packages installed - just install any that are missing.

2. **Install SCons**

	The C++ build system uses SCons. On Suse, this is available via normal package install. On Redhat, download and install package manually:

   On RHEL6 and RHEL7, download scons-2.2.0-1.noarch.rpm from [SCons 2.2.0 Downloads](http://sourceforge.net/projects/scons/files/scons/2.2.0) and install it like this:

        wget http://sourceforge.net/projects/scons/files/scons/2.2.0/scons-2.2.0-1.noarch.rpm
        sudo rpm -i scons-2.2.0-1.noarch.rpm

#Build and Install the C++ Driver

1. ***Download the Driver***

    Clone the C++ driver source code:

    ```shell
    cd <working-path>
    git clone https://github.com/mongodb/mongo-cxx-driver
    cd mongo-cxx-driver
    git checkout legacy-1.0.5
    ```
    
2. ***Build and Install***
    
    Use SCons to build and install it:

    ```shell
    sudo scons --prefix=/usr/local install
    ```
    
#Basic validation test

The example code section given below is used to perform a basic test to ensure that the MongoDB C++ Driver is working as expected, and can connect to, modify and query a MongoDB server.

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
    
    Use `vi` to create test.cxx with the content shown below.  This code connects to a MongoDB server, inserts some documents and then queries the database to read them back and display them. 
	
	_**Remember**_, if you are connecting to a remote server then you need to substitute the `localhost` with the hostname or IP address of the MongoDB server.
    
    ```shell
    #include <cstdlib>
    #include <iostream>
    #include "mongo/client/dbclient.h" // for the driver
    #include "mongo/bson/bson.h" // for BSON datatypes

    using mongo::BSONObj;
    using mongo::BSONObjBuilder;
    using mongo::DBClientCursor;

    const char *server("localhost");
    const char *db_collection("ibm_test_db.mongodb_cpp_test");

    int main() {
        mongo::DBClientConnection db;

        mongo::client::initialize();
        try {
            db.connect(server);
            std::cout << "connected ok" << std::endl;
        } catch( const mongo::DBException &e ) {
            std::cout << "caught connect error: " << e.what() <<     std::endl;
        }

        mongo::OID oid = mongo::OID::gen();
        BSONObjBuilder b;
        b.append("_id", oid);
        b.append("company", "IBM");
        b.append("project", "MongoDB Driver");
        b.append("language", "C++");
        b.append("version", "legacy");
        BSONObj p = b.obj();


        db.dropCollection(db_collection, NULL);

        try {
            db.insert(db_collection, p);
            std::cout << "inserted ok" << std::endl;
        } catch( const mongo::DBException &e ) {
            std::cout << "caught insert error: " << e.what() << std::endl;
        }

        for ( int i = 0; i < 3 ; i++) {
            mongo::OID oid = mongo::OID::gen();
            BSONObjBuilder b;
            b.append("_id", oid);
            b.append("line", i);
            BSONObj p = b.obj();
            db.insert(db_collection, p);
        }

        try {
            std::auto_ptr<DBClientCursor> cursor = db.query  (db_collection, BSONObj() );
            std::cout << "find ok" << std::endl;
            while (cursor->more()) {
                std::cout << cursor->next().toString() << std::endl;
            }
        } catch( const mongo::DBException &e ) {
            std::cout << "caught find error: " << e.what() << std::endl;
        }
 
        return EXIT_SUCCESS;
    }
    ```
    
3. ***Compile*** 

    Compile and run the test program by:
    
    ```shell
        g++ test.cxx -o test -pthread -lmongoclient -lboost_thread-mt -lboost_system -lboost_regex
        ./test
    ```
    
    Executing the script should produce output similar to this (the Object Ids will vary, but typically will be consecutive):
    
    ```shell
    { _id: ObjectId('55f839498d772db54d86f38c'), company: "IBM", project: "zLinux", driver: "C++", version: "legacy" }
    { _id: ObjectId('55f8394a8d772db54d86f38d'), row: 0 }
    { _id: ObjectId('55f8394a8d772db54d86f38e'), row: 1 }
    { _id: ObjectId('55f8394a8d772db54d86f38f'), row: 2 }
    ```