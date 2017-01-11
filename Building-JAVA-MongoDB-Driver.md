# Building MongoDB Java Driver


The instructions provided below specify the steps to build MongoDB Java Driver 3.4.0 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2, SLES 11-SP4 and Ubuntu 16.04/16.10.

_**General Notes:**_  
_* When following the steps below please use a standard permission user unless otherwise specified_  
_* A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_  

### Prerequisites
  * IBM Java 8 SDK binary (SLES 11-SP4 & UBUNTU 16.04/16.10)
  -- Download IBM Java 8 SDK binary from [IBM Java 8](http://www.ibm.com/developerworks/java/jdk/linux/download.html) and follow the instructions as per given in the link

#### Step 1: Install the Dependencies
* RHEL 6.8

        sudo yum install -y git java-1.8.0-ibm-devel.s390x  
   
* RHEL 7.1/7.2/7.3

        sudo yum install -y java-1.8.0-ibm-devel.s390x   
   
* SLES 11-SP4

        sudo zypper install -y git
 
* SLES 12-SP1/12-SP2

        sudo zypper install -y git java-1_8_0-ibm-devel
                                   
* Ubuntu 16.04/16.10
		
        sudo apt-get install -y git
        
#### Step 2: Build and install MongoDB Java Driver 
* Download the source tree and checkout the required version

        cd /<source_root>/
        git clone https://github.com/mongodb/mongo-java-driver.git
        cd /<source_root>/mongo-java-driver
        git checkout r3.4.0

* Start the build process

        ./gradlew assemble

    _**Note:** This will automatically download gradle and a number of other requirements.Occasionally gradlew may not be able to find `java`, to resolve this `export JAVA_HOME=/usr/lib/jvm/java-1.8.0` on RHEL 7.1/7.2/7.3 platform and `export JAVA_HOME=/usr/lib64/jvm/java-1.8.0` on SLES 12-SP1/12-SP2 platform_
    
* Run test cases (Optional)

        ./gradlew check 
    

    _**Note:** The MongoDB Driver needs access to a running MongoDB server, either on your local server or a remote system._
    _If MongoDB server is running on remote machine, pass a connection string/argument -Dorg.mongodb.test.uri=mongodb://example.com:27017/ while running test cases_  
    

### Basic validation test
    
The example code section given below is used to perform a basic test to ensure that the MongoDB Java Driver is working as expected, and can connect to, modify and query a MongoDB server.

* Prerequisites

    The MongoDB Driver needs access to a running MongoDB server, either on your local server or a remote system. The following commands are an example of how to start up a MongoDB server and then connect to it with the client shell, but note that MongoDB has not been installed as part of these instructions, and typically you would be running MongoDB on a remote server.

    ```shell
    mongod  --setParameter textSearchEnabled=true > /tmp/mongodb.log &
    mongo --host localhost 
    ```
    Which would typically give a command prompt such as:
    
    ```shell
    MongoDB shell version: 2.4.10 connecting to: localhost:27017/test > 
    ```
    The example code below will need to be modified to use your remote server hostname or IP address instead of "localhost", if you are attempting to connect to your own (remote) server.
    
* The Test Code
    
    Create a file named test.java with the content shown below. This code connects to a MongoDB server, inserts some documents and then queries the database to read them back and display them. 
	
	_**Remember**_, if you are connecting to a remote server then you need to substitute the `localhost` with the hostname or IP address of the MongoDB server.

    ```java
    import java.util.ArrayList;
    import java.util.List;
    import org.bson.Document;
    import com.mongodb.MongoClient;
    import com.mongodb.client.MongoCollection;
    import com.mongodb.client.MongoCursor;
    import com.mongodb.client.MongoDatabase;


    public class test {
        public static void main(String[] args) {

            String server_name="localhost";
            String database_name="ibm_test_db";
            String collection_name="mongodb_java_driver";

            MongoClient mongoClient = new MongoClient(server_name);
            MongoDatabase database = mongoClient.getDatabase(database_name);
            MongoCollection<Document> collection = database.getCollection(collection_name);

            collection.drop();

            Document doc = new Document("company", "IBM")
                .append("MongoDB Driver", new Document("language", "Java").append("version", "r3.4.0"));
            collection.insertOne(doc);

            List<Document> documents = new ArrayList<Document>();
            for (int i = 0; i < 3; i++) {
                documents.add(new Document("line", i));
            }
            collection.insertMany(documents);

            Document myDoc = collection.find().first();
            System.out.println(myDoc.toJson());
            MongoCursor<Document> cursor = collection.find().iterator();
            try {
                cursor.next();
                while (cursor.hasNext()) {
                    System.out.println(cursor.next().toJson());
                }
            } finally {
                cursor.close();
            }
        }
    }
    ```
     
*  Execute
    
    Compile and run the test program by:

    ```shell
    javac -cp /<source_root>/mongo-java-driver/mongo-java-driver/build/libs/mongo-java-driver-3.4.0.jar test.java
    java -cp /<source_root>/mongo-java-driver/mongo-java-driver/build/libs/mongo-java-driver-3.4.0.jar:. test
    ```
    Executing the test program should produce output similar to this (the Object Ids will vary, but typically will be consecutive):

    ```shell
    { "_id" : { "$oid" : "560562bf35203f023ee2f7db" }, "company" : "IBM", "MongoDB Driver" : { "language" : "Java", "version" : "r3.4.0" } }
	{ "_id" : { "$oid" : "560562bf35203f023ee2f7dc" }, "line" : 0 }
	{ "_id" : { "$oid" : "560562bf35203f023ee2f7dd" }, "line" : 1 }
	{ "_id" : { "$oid" : "560562bf35203f023ee2f7de" }, "line" : 2 }
    ```

### References:
[Java MongoDB Driver Document](https://docs.mongodb.com/ecosystem/drivers/java/)

[MongoDB](https://www.mongodb.com/)
