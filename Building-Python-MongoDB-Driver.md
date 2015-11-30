The Python driver for MongoDB (PyMongo) can be built for Linux on z Systems running RHEL 6.6, RHEL 7.1, SLES 11 and SLES 12 by following these instructions. Version 3.0.3 of PyMongo has been successfully built & tested this way.

_**General Notes:**_ 	

i) _When following the steps below please use a standard permission user unless otherwise specified._
	 
ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

iii) _Where the instructions refer to 'vi' you may, of course, use an editor of your choice._

## Building PyMongo 3.0.3

1. Install standard utilities and platform specific dependencies :

   RHEL 6.6/RHEL 7.1
   ```shell
   sudo yum install git 
   ``` 
   SLES 11/SLES 12
   ```shell
   sudo zypper install git python-xml
   ```
2. Create a working directory with write permission to use as an installation workspace:

   ```shell
   mkdir /<source_root>/
   cd /<source_root>/
   ```

3. Check-out PyMongo source from github.  It will be placed in `/<source_root>/pymongo` :

   ```shell
   git clone git://github.com/mongodb/mongo-python-driver.git pymongo
   cd pymongo
   git checkout 3.0.3
   ```

4. Configure and install pymongo :

   ```shell
   sudo python setup.py install
   ```
5. Verify if pymongo has installed successfully :

   ```shell
	python
   ```
	You should get something like the following result:

   ```shell
	Python 2.3.4 (#1, Jan  9 2007, 16:40:09)
	[GCC 3.4.6 20060404 (Red Hat 3.4.6-3)] on linux2
	Type "help", "copyright", "credits" or "license" for more 	information.
	>>>
   ```
   Now verify that pymongo has been installed:

   ```shell
	>>> import pymongo
   ```
	If you get no errors from this import statement, then PyMongo has been installed successfully.

### Basic validation test
    
The example code section given below is used to perform a basic test to ensure that the MongoDB Python Driver is working as expected, and can connect to, modify and query a MongoDB server.

1. ***Prerequisites***

    The MongoDB Driver needs access to a running MongoDB server, either on your local server or a remote system. The following commands are an example of how to start up a MongodDB server and then connect to it with the client shell, but note that MongoDB has not been installed as part of these instructions, and typically you would be running MongoDB on a remote server.

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
    
    Create a file named test.py with the content shown below.  This code connects to a MongoDB server, inserts some documents and then queries the database to read them back and display them. 
	
	_**Remember**_, if you are connecting to a remote server then you need to substitute the `localhost` with the hostname or IP address of the MongoDB server.

    ```shell
    import pprint
    import pymongo

    from pymongo import MongoClient

    server="localhost";
    database="ibm_test_db";
    collection="mongodb_python_driver";

    serverdb="mongodb://" + server + ":27017/";

    client = MongoClient(serverdb)
 
    db = client[database];

    db[collection].drop();

    header = {"company": "IBM",
              "project": "MongoDB Driver",
              "language": "python",
              "version": "3.0.3"};

    db[collection].insert_one(header);

    for i in range (0, 3):
            doc = {"line": i};
            db[collection].insert_one (doc);

    for gotdoc in db[collection].find():
            pprint.pprint(gotdoc);

    ```								

3. ***Run Test Script*** 
   
    ```shell
    python test.py
    ```
    
    The output from the above should look similar to:
	
    ```shell
    {u'_id': ObjectId('560eb1ff051ba90001d927a0'),
 u'company': u'IBM',
 u'language': u'python',
 u'project': u'MongoDB Driver',
 u'version': u'3.0.3'}
    {u'_id': ObjectId('560eb1ff051ba90001d927a1'), u'line': 0}
    {u'_id': ObjectId('560eb1ff051ba90001d927a2'), u'line': 1}
    {u'_id': ObjectId('560eb1ff051ba90001d927a3'), u'line': 2}
    ```