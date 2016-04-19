The [MongoDB Java Driver](http://docs.mongodb.org/ecosystem/drivers/java/) is a Java program, and as such version 3.2.0 can either be downloaded directly or built for a Linux on z System running RHEL 7.1/6.6 or SLES 12/11 or Ubuntu 16.04.

If you wish to rebuild the jar please follow the instructions below:

_**General Notes:**_

_i) When following the steps below please use a standard permission user unless otherwise specified._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_  
_iii) For convenience `vi` has been used in the instructions below when editing files, replace with your desired editing program if required._


1. Install the build dependencies

    On RHEL 7.1 and 6.6 systems:
    ```shell
    sudo yum install git java-1.7.1-ibm-devel
    ```
    On SLES 11 systems:
    ```shell
    sudo zypper install -y git java-1_7_0-ibm-devel-1.7.0_sr9.10-9.1.s390x
    ```
    On SLES 12 systems:
    ```shell
    sudo zypper install -y git java-1_7_1-ibm-devel
    ```
    On Ubuntu 16.04 systems:
    ```shell
    sudo apt-get update
    sudo apt-get install git openjdk-8-jdk
    ```
  Some of these packages may have been installed already - just install any that are missing.
  
2. Download the source tree and checkout the required version

    ```shell
    cd /<source_root>/
    git clone https://github.com/mongodb/mongo-java-driver.git
    cd /<source_root>/mongo-java-driver
    git checkout r3.2.0
    ```
3. Start the build process

    ```shell
    ./gradlew assemble
    ```
    _**Note:** This will automatically download gradle and a number of other requirements_  
    _**Note:** Occasionally gradlew may not be able to find `java`, to resolve this `export JAVA_HOME=/usr/lib/jvm/java` on RHEL platforms and `export JAVA_HOME=/usr/lib64/jvm/java` on SLES platforms_
4. _**Optionally**_ run all the tests

    Correct a relative path to be absolute and then run all available tests
    ```shell
    sed 's\config/checkstyle-exclude.xml\/<source_root>/mongo-java-driver/config/checkstyle-exclude.xml\' config/checkstyle.xml > config/checkstyle.xml.new
    mv -f config/checkstyle.xml.new config/checkstyle.xml
    ./gradlew check
    ```
    _**Note:** This will require a fairly powerful machine and access to MongoDB to complete successfully_  
    Additional information about the tasks available to gradle can be found via `./gradlew tasks` which will list the available tasks, more information can be found on [docs.gradle.org](https://docs.gradle.org/current/userguide/java_plugin.html) or in the `build.gradle` files in `/<source_root>/mongo-java-driver/` and sub-directories.

### Basic validation test
    
The example code section given below is used to perform a basic test to ensure that the MongoDB Java Driver is working as expected, and can connect to, modify and query a MongoDB server.

1. ***Prerequisites***

    The MongoDB Driver needs access to a running MongoDB server, either on your local server or a remote system. The following commands are an example of how to start up a MongoDB server and then connect to it with the client shell, but note that MongoDB has not been installed as part of these instructions, and typically you would be running MongoDB on a remote server.

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
                .append("MongoDB Driver", new Document("language", "Java").append("version", "r3.2.0"));
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
     
3. ***Execute*** 
    
    Compile and run the test program by:

    ```shell
    javac -cp /<source_root>/mongo-java-driver/mongo-java-driver/build/libs/mongo-java-driver-3.2.0.jar test.java
    java -cp /<source_root>/mongo-java-driver/mongo-java-driver/build/libs/mongo-java-driver-3.2.0.jar:. test
    ```
    Executing the test program should produce output similar to this (the Object Ids will vary, but typically will be consecutive):

    ```shell
    { "_id" : { "$oid" : "560562bf35203f023ee2f7db" }, "company" : "IBM", "MongoDB Driver" : { "language" : "Java", "version" : "r3.2.0" } }
	{ "_id" : { "$oid" : "560562bf35203f023ee2f7dc" }, "line" : 0 }
	{ "_id" : { "$oid" : "560562bf35203f023ee2f7dd" }, "line" : 1 }
	{ "_id" : { "$oid" : "560562bf35203f023ee2f7de" }, "line" : 2 }
    ```