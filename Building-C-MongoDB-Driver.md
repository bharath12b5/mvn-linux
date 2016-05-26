The C MongoDB Driver can be built for Linux on z Systems running SLES 12, SLES 11, RHEL 7, RHEL 6 and Ubuntu 16.04 by following these instructions.  Versions 1.3.x has been successfully built & tested this way.

_**General Notes:**_ 	

i) When following the steps below please use a standard permission user unless otherwise specified.

ii) Where the instructions refer to 'vi' you may, of course, use an editor of your choice

iii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it.

iv) Version 1.3.x refers to the current stable version of 1.3 branch releases. At the time of writing recipe, it was 1.3.2.

# Building MongoDB C Driver

1. **Install build dependencies**

  For **SLES 12**
  ```
  sudo zypper install -y autoconf automake gcc libtool make wget
  ```
  
  For **SLES 11**
  ```
  sudo zypper install -y autoconf automake gcc libtool make pkg-config tar wget
  ```
  
  For **RHEL 7.1**
  ```
  sudo yum install -y autoconf automake gcc libtool make which wget
  ```
  
  For **RHEL 6.6**
  ```
  sudo yum install -y autoconf automake gcc libtool make tar which wget
  ```
  
  For **Ubuntu 16.04**
  ```
  sudo apt-get update
  sudo apt-get install autoconf automake gcc libtool make wget pkg-config
  ```
 
  You may already have these packages installed - just install any that are missing.
  
  
2. **Download and configure the C Driver**

   For version 1.3.x of the driver:
 
  ```
    cd /<source_root>/
	wget -O mongo-c-driver-1.3.x.tar.gz  https://github.com/mongodb/mongo-c-driver/releases/download/1.3.x/mongo-c-driver-1.3.x.tar.gz
	tar xzf mongo-c-driver-1.3.x.tar.gz
	cd mongo-c-driver-1.3.x
	./configure
  ```

3. **Build and install the Driver**

    ```
    make && sudo make install
    ```
    
### Basic validation test
    
The example code section given below is used to perform a basic test to ensure that the MongoDB C Driver is working as expected, and can connect to, modify and query a MongoDB server.

1. ***Prerequisites***

    The MongoDB Driver obviously needs access to a running MongoDB server, either on your local server or a remote system. The following commands are an example of how to start up a MongoDB server and then connect to it with the client shell, but note that MongoDB has not been installed as part of these instructions, and typically you would be running MongoDB on a remote server.

    ```
    mongod > /tmp/mongodb.log &
    mongo --host localhost 
    ```
    Which would typically give a command prompt such as:
    
    ```
    MongoDB shell version: 2.4.10 connecting to: localhost:27017/test > 
    ```
    The example code below will need to be modified to use your remote server hostname or IP address instead of "localhost", if you are attempting to connect to your own (remote) server.
    
2. ***The Test Code***
    
    Create a file named test.c with the content shown below.  This code connects to a MongoDB server, inserts some documents and then queries the database to read them back and display them. 
	
	_**Remember**_, if you are connecting to a remote server then you need to substitute the `localhost` with the hostname or IP address of the MongoDB server.

    ```
    #include <bson.h> 
    #include <mongoc.h> 
    #include <stdio.h> 

    const char* server="localhost";
    const char* test_db="ibm_test_db";
    const char* collection_name="mongodb_c_driver";

    int
    main (int argc,
          char *argv[])
    {
        mongoc_client_t *client;
        mongoc_collection_t *collection;
        mongoc_cursor_t *cursor;
        bson_error_t error;
        bson_oid_t oid;
        bson_t *doc, query;
        const bson_t *gotdoc;
        char *str;
        int i;
        char istr[2], server_addr[50];

        sprintf(server_addr, "mongodb://%s:27017/", server);

        mongoc_init ();

        client = mongoc_client_new (server_addr);

        collection = mongoc_client_get_collection (client,     test_db, collection_name);

        if (!mongoc_collection_drop (collection, &error)) {
            printf ("%s\n", error.message);
        }


        doc = bson_new ();
        BSON_APPEND_UTF8 (doc, "company", "IBM");
        BSON_APPEND_UTF8 (doc, "project", "MongoDB Driver");
        BSON_APPEND_UTF8 (doc, "language", "C");
        BSON_APPEND_UTF8 (doc, "version", "1.2.0-dev");

        if (!mongoc_collection_insert (collection, MONGOC_INSERT_NONE, doc, NULL, &error)) {
            printf ("%s\n", error.message);
        }
        bson_destroy (doc);

        for ( i = 0; i < 3; i++) {
            doc = bson_new ();
            sprintf (istr, "%d", i);
            BSON_APPEND_UTF8 (doc, "line", istr);

            if (!mongoc_collection_insert (collection, MONGOC_INSERT_NONE, doc, NULL, &error)) {
                printf ("%s\n", error.message);
            }
            bson_destroy (doc);
        }

        bson_init (&query);

        cursor = mongoc_collection_find (collection, MONGOC_QUERY_NONE, 0, 0, 0, &query, NULL, NULL);

        while (mongoc_cursor_next (cursor, &gotdoc)) {
                str = bson_as_json (gotdoc, NULL);
            fprintf (stdout, "%s\n", str);
            bson_free (str);
        }

        bson_destroy (&query);

        mongoc_cursor_destroy (cursor);
        mongoc_collection_destroy (collection);
        mongoc_client_destroy (client);

        return 0;
    }
    ```
	
3. ***Execute*** 
    
  Compile and run the test program by:

    ```
    export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/
    gcc test.c -o test $(pkg-config --cflags --libs libmongoc-1.0)
    export LD_LIBRARY_PATH=/usr/local/lib
    ./test
    ```
  Executing the test program should produce output similar to this (the Object Ids will vary, but typically will be consecutive):

    ```
    { "_id" : { "$oid" : "55f84c94646895004533b171" }, "company" : "IBM", "project" : "zLinux", "driver" : "C", "version" : "1.2.0" }
	{ "_id" : { "$oid" : "55f84c94646895004533b172" }, "row" : "0" }
	{ "_id" : { "$oid" : "55f84c94646895004533b173" }, "row" : "1" }
	{ "_id" : { "$oid" : "55f84c94646895004533b174" }, "row" : "2" }
    ```