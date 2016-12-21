# Building Python MongoDB Driver

The instructions provided below specify the steps to build Python MongoDB Driver (PyMongo) version 3.4.0 on IBM z Systems for RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	

* _When following the steps below please use a standard permission user unless otherwise specified._
	 
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

##Step 1: Building and Installing PyMongo
####1.1) Install dependencies

   * RHEL 6.8/7.1/7.2/7.3
   ```shell
   sudo yum install git openssl openssl-devel pyOpenSSL
   ``` 

   * SLES 11-SP4
   ```shell
   sudo zypper install git python-xml python-openssl openssl-devel openssl
   ```
   * SLES 12/12-SP1/12-SP2
   ```shell
   sudo zypper install git python-xml
   ```
   
   * Ubuntu 16.04/16.10
   ```shell
   sudo apt-get update
   sudo apt-get install git openssl libssh-dev python python-openssl python-setuptools
   sudo easy_install pip  
   ```
   
####1.2) Create a working directory with write permission to use as an installation workspace

   ```shell
   mkdir /<source_root>/
   cd /<source_root>/
   ```

####1.3) Download source code

   ```shell
   git clone git://github.com/mongodb/mongo-python-driver.git pymongo
   cd pymongo
   git checkout 3.4.0
   ```

####1.4) Configure and install

   ```shell
   sudo python setup.py install
   ```
   
####1.5) Execute test cases 

   Access to MongoDB Server is required to execute test cases.
 
   * If MongoDB Server is running on same machine then execute the following:
   ```shell
   sudo python setup.py test
   ```
   
   * If MongoDB Server is running on remote machine, then create a `test_cases.sh` file with following content:
   ```shell
   export DB_IP=<MongoDB_Server_IP>
   python setup.py test
   ```
   * Give execute permission to `test_cases.sh` file and execute it as follows:
   ```shell
   chmod +x test_cases.sh
   sudo ./test_cases.sh
   ```
 
  **Note: Execute the below steps only in case of test case failures**
    
   * If test_bad_encode test case fails with error: "TypeError: encoder expected a mapping type but got: {'a': {...}}", or if  test_get_last_version test case fails with AssertionError then it might be an issue with Python version. Try installing Python 2.7.9 using following steps and re-run the test cases.
  
   #### Install dependencies as follows
   
   * RHEL 6.8/7.1/7.2/7.3
   
   ```shell
   sudo yum install tar xz wget zlib-devel
   ``` 
   
   * SLES 11-SP4/12/12-SP1/12-SP2
   
   ```shell
   sudo zypper install tar xz wget zlib-devel
   ```
   
   * Follow the steps in [Python recipe](https://github.com/linux-on-ibm-z/docs/wiki/Building-Python-2.7.9) to install Python
   
   * If MongoDB Server is running on same machine then set the build location in path variable
   ```shell
   export PATH=<python-build-location>/bin:$PATH
   ```
   
   * If MongoDB Server is running on remote machine then  put the "export PATH=<python-build-location>/bin:$PATH" in first line of test_cases.sh file.
   
   ```shell
   export PATH=<python-build-location>/bin:$PATH
   export DB_IP=<MongoDB_Server_IP>
   python setup.py test
   ```
Re-run test cases as mentined above.
   
####1.6) Verify installation 

   ```shell
	python
   ```
   You should get something like the following result

   ```shell
	Python 2.3.4 (#1, Jan  9 2007, 16:40:09)
	[GCC 3.4.6 20060404 (Red Hat 3.4.6-3)] on linux2
	Type "help", "copyright", "credits" or "license" for more 	information.
	>>>
   ```
   Now verify that PyMongo has been installed

   ```shell
	>>> import pymongo
   ```
   If you get no errors from this import statement, then PyMongo has been installed successfully.To exit from python prompt either use ctrl + d or type exit(). 

## Step 2:Basic validation test
    
The example code section given below is used to perform a basic test to ensure that the Python MongoDB Driver is working as expected, and can connect to, modify and query a MongoDB server.

####2.1) Prerequisites

  Python MongoDB Driver needs access to a running MongoDB server, either on your local server or a remote system. The following commands are an example of how to start up a MongoDB server and then connect to it with the client shell, but note that MongoDB has not been installed as part of these instructions, and typically you would be running MongoDB on a remote server.

  ```shell
  mongod > /tmp/mongodb.log &
  mongo --host localhost 
  ```
    
  Which would typically give a command prompt such as
    
  ```shell
  MongoDB shell version: 2.4.10 connecting to: localhost:27017/test > 
  ```
    
  The example code below will need to be modified to use your remote server hostname or IP address instead of "localhost", if you are attempting to connect to your own (remote) server.
    
####2.2) The Test Code
    
  Create a file named test.py with the content shown below. This code connects to a MongoDB server, inserts some documents and then queries the database to read them back and display them. 
	
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
        "version": "3.4.0"};

  db[collection].insert_one(header);

  for i in range (0, 3):
      doc = {"line": i};
      db[collection].insert_one (doc);

  for gotdoc in db[collection].find():
      pprint.pprint(gotdoc);

  ```								

####2.3) Run Test Script 
   
  ```shell
  python test.py
  ```
    
  The output from the above should look similar to:
	
  ```shell
  {u'_id': ObjectId('560eb1ff051ba90001d927a0'),
  u'company': u'IBM',
  u'language': u'python',
  u'project': u'MongoDB Driver',
  u'version': u'3.4.0'}
  {u'_id': ObjectId('560eb1ff051ba90001d927a1'), u'line': 0}
  {u'_id': ObjectId('560eb1ff051ba90001d927a2'), u'line': 1}
  {u'_id': ObjectId('560eb1ff051ba90001d927a3'), u'line': 2}
  ```

##References:
* [Python MongoDB installation ](http://api.mongodb.com/python/current/installation.html)
