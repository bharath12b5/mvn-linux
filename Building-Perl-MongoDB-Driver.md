The Perl MongoDB Driver v0.708.2.0 can be built for Linux on z Systems running RHEL 6.6/7.1 and SLES 11/12 by following these instructions.  Version  v0.708.2.0 has been successfully built and tested this way.  More information on MongoDB_Perl_Driver is available at http://docs.mongodb.org/ecosystem/drivers/perl/ and the source code can be obtained from the Perl code repository, CPAN, at http://search.cpan.org/dist/MongoDB/
.

_**General Notes:**_

i) _**Note:** This recipe provides the required Perl MongoDB Driver Linux on z Systems pre-requisites, a MongoDB Test script and directs the user to install a Perl module using tools from the Perl code repository CPAN.  The guidance to install the Perl module given here should work in most circumstances, but as details of a CPAN download are environment specific, the reader is referred documentation on the CPAN site (etc.) for further details._

ii) _**Note:** When following the steps below please use a standard permission user unless otherwise specified._

iii) _**Note:** This recipe specifies that CPAN commands should be run as `sudo` to ensure sufficient privileges to install Perl modules.  Depending on the environment employed, it may be possible to run the CPAN commands as a standard user._

iv) _**Note:** A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

## Building Perl MongoDB Driver

### Obtain pre-built dependencies.

1. Use the following commands to obtain dependencies :

   For RHEL
   ```shell
   sudo yum install make gcc tar zip perl perl-YAML cpan
   ```

   For SLES (Note : On SLES CPAN is bundled with perl)
   ```shell
   sudo zypper install  make gcc tar zip perl perl-YAML
   ```

### **OPTIONAL** Dependency Build - Install latest CPAN.

Recommended - Update CPAN (Currently upgrade to  version 2.10) and also two modules to help the Build/Make process.

   1. Check the version of CPAN that is installed.
      ```shell
      perl -MCPAN -le 'print "CPAN Version -> $CPAN::VERSION"'
      ```

   1. Install the latest CPAN version - (The -fi flags force install to over-ride some failures).
      ```shell
      sudo cpan -fi Bundle::CPAN
      ```
      i) _**Note:** CPAN will ask for some configuration questions the first time it is used.  Answers to these questions will vary dependent on the user's location and environment. CPAN is however generally able to 'tune' itself for an environment, and it is usually sufficient to accept the default answer to the questions offered.  In the event of difficulty please see the CPAN documentation link in the references below._

      ii) _**Note:** Once the CPAN configuration is complete the installation of `Bundle::CPAN` will begin .  Dependent on the software already installed, some Perl modules will likely need to be installed or updated. CPAN will ask to follow sub-dependencies automatically, store data on the system permanently, and will report any test failures (etc.). A working system will generally be obtained by accepting the default answers offered but keeping a note of the errors reported is recommended as they might be useful in later testing._

   1. Confirm the version of CPAN  has been updated correctly.
      ```shell
      perl -MCPAN -le 'print "CPAN Version -> $CPAN::VERSION"'
      ```

   1. These two modules are bundled with the Perl install - It makes sense bring them up to the latest versions.
      ```shell
      sudo cpan Config::AutoConf
      sudo cpan -fi Path::Tiny
      ```

      _**Note:** At the time of writing `Path::Tiny` failed a test on RHEL7.1 - The '-fi' options help force an install._

### Product Build - MongoDB_Perl_Driver.

   1. Use CPAN to install MongoDB, (Respond to dialog questions as they occur by accepting the default).
      ```shell
      sudo cpan MongoDB
      ```
      i) _**Note:** The CPAN dialog for the MongoDB module install follows the pattern described above for `Bundle::CPAN`. In the first instance to accept the default answers offered, taking note of any failures._

      ii) _**Note:** The CPAN repository is subject to continual updates which leads to occasional intermittent build, and test failures, possibly leading to a dependent module not being installed. Where a module has not been installed, it will first become apparent at run time, or in the testing (below) should a script attempt to access missing code. In such circumstances you can, try to reload the failed module with `sudo cpan <moduleName>`, or find the CPAN build directory (default=`/root/.cpan/build/`) to install it 'by hand', or try to install an older/later module version, or consult the documentation or 'the web' for further advice._

      iii) _**Note:** On completion more than 100 Perl modules may have been loaded into the CPAN build directory and installed on the system._

### Post Installation  Testing.

   This testing is based on information at https://metacpan.org/pod/distribution/MongoDB/lib/MongoDB/Tutorial.pod and demonstrates a MongoDB connection to a remote host.  A script (shown below) connects to the host and port (must be set to suit the target environment), and uses a database 'tutorial' and collection 'users' (which can also be edited). Information on MongoDB Installation is available at http://docs.mongodb.org/manual/installation/ .

   The script adds two users to the collection, one of whom likes 'math' and consequently is a geek.  There are some searches on the data, and then the other user is also made a geek, and the data searched again. A sample of the expected output for this is shown below.

   Data in the database accumulates if the script is run multiple times.  To aid testing, a Perl variable '$dropdata' can be set to '1', so the database will be deleted on exit. (Please only make this change where it can only affect this test database, otherwise important data can be lost).  In addition,  more detailed output can be obtained by setting the Perl variable '$dumper = 1', and sample output for this is also shown below.

   At run time, if a required module is found missing, a message of the form `Can't locate <Directory>>/<Module>.pm in @INC (@INC contains: inc /usr/...` is issued. Installation of the missing module can then be attempted using CPAN commands described above, (perhaps using the `-fi` flags to help force installation).

   To perform the MongoDB tests proceed as follows:-

   1. Create a `/<source_root>/` directory (mentioned in a General Note above), and `cd` into it .
      ```shell
      mkdir /<source_root>/
      cd  /<source_root>/
      ```

   1. Cut and paste the script (below) into a text file named `/<source_root>/test_MongoDB.pl`

   1. Modify the Perl variables $DBhost, $DBport, $DBname, and $DBcolletion to connect with the target MongoDB database, then save the file.

   1. Ensure execute permissions are set on the file so it can be run.
      ```shell
      chmod 755 /<source_root>/test_MongoDB.pl
      ```

   1. Check the changes made to the Perl script are valid (Response should be `.../test_MongoDB.pl syntax OK`).
      ```shell
      perl -wc /<source_root>/test_MongoDB.pl
      ```

   1. Run the script, and compare the results with those shown below.
      ```shell
      perl /<source_root>/test_MongoDB.pl
      ```

   1. **OPTIONAL** Make any desired changes to the script and then remove the ` /<source_root>/` directory to tidy up.
      ```shell
      rm -rf  /<source_root>/
      ```

### Reference Files used for Testing :

   Perl script for 'cut and paste' into text file 'test_MongoDB.pl'.

```shell
#!/usr/bin/perl
use strict ;
use MongoDB ;
use MongoDB::OID; ;
use Data::Dumper ;
#
# This is a simple test script to demonstrate  MongoDB accessed via a Perl driver. The code is based on information
# available at the below link, and the user is directed there for further explanation :

# https://metacpan.org/pod/distribution/MongoDB/lib/MongoDB/Tutorial.pod


# Configuration variables  (Need to be set to match the existing installation)
# ----------------------------------------------------------------------------
my $DBhost = 'SomeHost' ;    # Configure - to the machine hosting MongoDB.
my $DBport = '27017' ;       # Configure - 27017 is the default host, but site specific configuration may be required.
my $DBname = 'tutorial' ;    # Configure - Should be OK to use 'as is' provided the name does not clash with any existing database
my $DBcollection = 'users' ; # Configure - Should be OK to use 'as is' provided the name does not clash with any existing collection

#  Note: - if variable $dropdata is set to a 'non-empty' value the database and collection will be deleted when this script
#          is run. *** Do not set this variable unless you are sure no important data can be deleted ***.
my $dropData = '0' ; # See Note:- (above)Set this to '1' to delete the collection on exit (so it is empty next time)

my $dumper = '0' ;   # Set this to '1' for detailed output
#
# Add some users to add to the collection - these data can be edited to suit.
# ----------------------------------------------------------------------------
my $u1 = {  "name" => "Eddy",
             "age" => 52,
           "likes" => [qw/skiing math ponies/]
        };
#
my $u2 = {"name" => "Stan" };

#
# Connect to the database_host, connect to the database, and set the collection
# -----------------------------------------------------------------------------
my $client = MongoDB::Connection->new(host => "$DBhost", port => "$DBport");
my $db = $client->get_database("$DBname");
my $users = $db->get_collection("$DBcollection" );

# Start the database test
# ----------------------------------------------------------------------------

print("\nAdd two users (Eddy and Stan) and use the Dumper module to print some Details:- \n") ;
print("----------------------------------------------------------------------------------\n");
my $id1 = $users->insert( $u1);
print 'Dumper id1 - $$u1{name} = '.Dumper $id1;

my $id2 = $users->insert( $u2);
print 'Dumper id2 - $$u2{name} = '.Dumper $id2;

print("\nGet a list of the users in the collection :- \n") ;
print("-----------------------------------------------\n") ;
my $all_users = $users->find;
if ( $dumper ) { print 'Dumper all-users = '.Dumper $all_users;}
while (my $all = $all_users->next) { print 'User:'.$all->{'name'}."\n"; }

print("\nGet the name for user user1 :- \n") ;
print("---------------------------------\n");
my $user1 = $users->find({"name" => "$$u1{name}"});
if ( $dumper ) { print 'Dumper user1 = '.Dumper $user1;}
while (my $user = $user1->next) { print 'User1 :'.$user->{'name'}."\n"; }

print("\nFind the geek who likes math (should be user1 only) :- \n") ;
print("---------------------------------------------------------\n");
my $geeks = $users->find({"likes" => "math"});
if ( $dumper ) { print 'Dumper geeks = '.Dumper $geeks ; }
while (my $geek = $geeks->next) { print 'Geek:'.$geek->{'name'}."\n"; }

print("\nMake user2 a geek and change the name accordingly :- \n") ;
print("-------------------------------------------------------\n") ;
print("update({\"_id\" => $id2}, {'push' => {'likes' => 'math'}})\n");
print("update({\"_id\" => $id2}, {'set' => {'name' => \"$$u2{name} the Geek\"}})\n") ;

$users->update({"_id" => $id2}, {'$push' => {'likes' => 'math'}});
$users->update({"_id" => $id2}, {'$set' => {'name' => "$$u2{name}".' the Geek'}});

# ... and list the Geeks again ( should now include 'Stan the geek')
print("\nSearch again for the geeks which should now include user2 :- \n") ;
print("---------------------------------------------------------------\n") ;
my $geek2 = $users->find({"likes" => "math"});
if ( $dumper ) { print 'Dumper '.$$u2{name}.' is now a Geek = '.Dumper $geek2 ; }
while (my $geek = $geek2->next) { print 'Geek:'.$geek->{'name'}."\n"; }
#
# *** WARNING *** This next code segment can delete the collection which is useful for testing
#                 purposes.  Be sure not to delete any important data.
if ( $dropData ) {
   my $dropDB = $db->drop;
   print("\n---------------------------------------------------------------\n") ;
   print "Dropped Database - '$DBname' = ".Dumper $dropDB;
   print("---------------------------------------------------------------\n") ;
}
exit ;
#End.
```

   Sample results that should be obtained from running the testMongoDb.pl script with the dumper flag set to off.
```shell
Add two users (Eddy and Stan) and use the Dumper module to print some Details:-
----------------------------------------------------------------------------------
Dumper id1 - $$u1{name} = $VAR1 = bless( {
                 'value' => '559cf20179e8d86f4956d611'
               }, 'MongoDB::OID' );
Dumper id2 - $$u2{name} = $VAR1 = bless( {
                 'value' => '559cf20179e8d86f4956d612'
               }, 'MongoDB::OID' );

Get a list of the users in the collection :-
-----------------------------------------------
User:Eddy
User:Stan

Get the name for user user1 :-
---------------------------------
User1 :Eddy

Find the geek who likes math (should be user1 only) :-
---------------------------------------------------------
Geek:Eddy

Make user2 a geek and change the name accordingly :-
-------------------------------------------------------
update({"_id" => 559cf20179e8d86f4956d612}, {'push' => {'likes' => 'math'}})
update({"_id" => 559cf20179e8d86f4956d612}, {'set' => {'name' => "Stan the Geek"}})

Search again for the geeks which should now include user2 :-
---------------------------------------------------------------
Geek:Eddy
Geek:Stan the Geek
```

   Sample results obtained from running the testMongoDb.pl script with the $dumper variable set to '1'.

```

Add two users (Eddy and Stan) and use the Dumper module to print some Details:- 
----------------------------------------------------------------------------------
Dumper id1 - $$u1{name} = $VAR1 = bless( {
                 'value' => '55c46945638587752f160621'
               }, 'MongoDB::OID' );
Dumper id2 - $$u2{name} = $VAR1 = bless( {
                 'value' => '55c46945638587752f160622'
               }, 'MongoDB::OID' );

Get a list of the users in the collection :- 
-----------------------------------------------
Dumper all-users = $VAR1 = bless( {
                 '_skip' => 0,
                 '_ns' => 'tutorial.users',
                 '_is_parallel' => 0,
                 '_master' => bless( {
                                       'w' => 1,
                                       'query_timeout' => 30000,
                                       'find_master' => 0,
                                       '_servers' => {},
                                       '_max_bson_wire_size' => 16793600,
                                       'sasl' => 0,
                                       'wtimeout' => 1000,
                                       'j' => 0,
                                       '_readpref_mode' => 0,
                                       'timeout' => 20000,
                                       '_readpref_pingfreq_sec' => 5,
                                       '_readpref_retries' => 3,
                                       '_is_mongos' => 0,
                                       'sasl_mechanism' => 'GSSAPI',
                                       'auto_connect' => 1,
                                       'auto_reconnect' => 1,
                                       'db_name' => 'admin',
                                       'ts' => 0,
                                       'ssl' => 0,
                                       'inflate_dbrefs' => 1,
                                       'port' => '27017',
                                       'host' => 'ww.xx.yy.zz',
                                       'inflate_regexps' => 0,
                                       'min_wire_version' => 0,
                                       'max_wire_version' => 3,
                                       'max_bson_size' => 16777216,
                                       'dt_type' => 'DateTime',
                                       '_max_write_batch_size' => 1000,
                                       '_opts' => {
                                                    'port' => '27017',
                                                    'host' => 'ww.xx.yy.zz'
                                                  }
                                     }, 'MongoDB::MongoClient' ),
                 'partial' => 0,
                 '_query' => bless( [
                                      {},
                                      [],
                                      [],
                                      0
                                    ], 'Tie::IxHash' ),
                 '_tailable' => 0,
                 '_client' => $VAR1->{'_master'},
                 '_limit' => 0,
                 '_agg_batch_size' => 0,
                 '_batch_size' => 0,
                 'slave_okay' => 0,
                 '_request_id' => 0,
                 'immortal' => 0,
                 'started_iterating' => 0
               }, 'MongoDB::Cursor' );
User:Eddy
User:Stan

Get the name for user user1 :- 
---------------------------------
Dumper user1 = $VAR1 = bless( {
                 '_skip' => 0,
                 '_ns' => 'tutorial.users',
                 '_is_parallel' => 0,
                 '_master' => bless( {
                                       'w' => 1,
                                       'query_timeout' => 30000,
                                       'find_master' => 0,
                                       '_servers' => {},
                                       '_max_bson_wire_size' => 16793600,
                                       'sasl' => 0,
                                       'wtimeout' => 1000,
                                       'j' => 0,
                                       '_readpref_mode' => 0,
                                       'timeout' => 20000,
                                       '_readpref_pingfreq_sec' => 5,
                                       '_readpref_retries' => 3,
                                       '_is_mongos' => 0,
                                       'sasl_mechanism' => 'GSSAPI',
                                       'auto_connect' => 1,
                                       'auto_reconnect' => 1,
                                       'db_name' => 'admin',
                                       'ts' => 0,
                                       'ssl' => 0,
                                       'inflate_dbrefs' => 1,
                                       'port' => '27017',
                                       'host' => 'ww.xx.yy.zz',
                                       'inflate_regexps' => 0,
                                       'min_wire_version' => 0,
                                       'max_wire_version' => 3,
                                       'max_bson_size' => 16777216,
                                       'dt_type' => 'DateTime',
                                       '_max_write_batch_size' => 1000,
                                       '_opts' => {
                                                    'port' => '27017',
                                                    'host' => 'ww.xx.yy.zz'
                                                  }
                                     }, 'MongoDB::MongoClient' ),
                 'partial' => 0,
                 '_query' => bless( [
                                      {
                                        'name' => 0
                                      },
                                      [
                                        'name'
                                      ],
                                      [
                                        'Eddy'
                                      ],
                                      0
                                    ], 'Tie::IxHash' ),
                 '_tailable' => 0,
                 '_client' => $VAR1->{'_master'},
                 '_limit' => 0,
                 '_agg_batch_size' => 0,
                 '_batch_size' => 0,
                 'slave_okay' => 0,
                 '_request_id' => 0,
                 'immortal' => 0,
                 'started_iterating' => 0
               }, 'MongoDB::Cursor' );
User1 :Eddy

Find the geek who likes math (should be user1 only) :- 
---------------------------------------------------------
Dumper geeks = $VAR1 = bless( {
                 '_skip' => 0,
                 '_ns' => 'tutorial.users',
                 '_is_parallel' => 0,
                 '_master' => bless( {
                                       'w' => 1,
                                       'query_timeout' => 30000,
                                       'find_master' => 0,
                                       '_servers' => {},
                                       '_max_bson_wire_size' => 16793600,
                                       'sasl' => 0,
                                       'wtimeout' => 1000,
                                       'j' => 0,
                                       '_readpref_mode' => 0,
                                       'timeout' => 20000,
                                       '_readpref_pingfreq_sec' => 5,
                                       '_readpref_retries' => 3,
                                       '_is_mongos' => 0,
                                       'sasl_mechanism' => 'GSSAPI',
                                       'auto_connect' => 1,
                                       'auto_reconnect' => 1,
                                       'db_name' => 'admin',
                                       'ts' => 0,
                                       'ssl' => 0,
                                       'inflate_dbrefs' => 1,
                                       'port' => '27017',
                                       'host' => 'ww.xx.yy.zz',
                                       'inflate_regexps' => 0,
                                       'min_wire_version' => 0,
                                       'max_wire_version' => 3,
                                       'max_bson_size' => 16777216,
                                       'dt_type' => 'DateTime',
                                       '_max_write_batch_size' => 1000,
                                       '_opts' => {
                                                    'port' => '27017',
                                                    'host' => 'ww.xx.yy.zz'
                                                  }
                                     }, 'MongoDB::MongoClient' ),
                 'partial' => 0,
                 '_query' => bless( [
                                      {
                                        'likes' => 0
                                      },
                                      [
                                        'likes'
                                      ],
                                      [
                                        'math'
                                      ],
                                      0
                                    ], 'Tie::IxHash' ),
                 '_tailable' => 0,
                 '_client' => $VAR1->{'_master'},
                 '_limit' => 0,
                 '_agg_batch_size' => 0,
                 '_batch_size' => 0,
                 'slave_okay' => 0,
                 '_request_id' => 0,
                 'immortal' => 0,
                 'started_iterating' => 0
               }, 'MongoDB::Cursor' );
Geek:Eddy

Make user2 a geek and change the name accordingly :- 
-------------------------------------------------------
update({"_id" => 55c46945638587752f160622}, {'push' => {'likes' => 'math'}})
update({"_id" => 55c46945638587752f160622}, {'set' => {'name' => "Stan the Geek"}})

Search again for the geeks which should now include user2 :- 
---------------------------------------------------------------
Dumper Stan is now a Geek = $VAR1 = bless( {
                 '_skip' => 0,
                 '_ns' => 'tutorial.users',
                 '_is_parallel' => 0,
                 '_master' => bless( {
                                       'w' => 1,
                                       'query_timeout' => 30000,
                                       'find_master' => 0,
                                       '_servers' => {},
                                       '_max_bson_wire_size' => 16793600,
                                       'sasl' => 0,
                                       'wtimeout' => 1000,
                                       'j' => 0,
                                       '_readpref_mode' => 0,
                                       'timeout' => 20000,
                                       '_readpref_pingfreq_sec' => 5,
                                       '_readpref_retries' => 3,
                                       '_is_mongos' => 0,
                                       'sasl_mechanism' => 'GSSAPI',
                                       'auto_connect' => 1,
                                       'auto_reconnect' => 1,
                                       'db_name' => 'admin',
                                       'ts' => 0,
                                       'ssl' => 0,
                                       'inflate_dbrefs' => 1,
                                       'port' => '27017',
                                       'host' => 'ww.xx.yy.zz',
                                       'inflate_regexps' => 0,
                                       'min_wire_version' => 0,
                                       'max_wire_version' => 3,
                                       'max_bson_size' => 16777216,
                                       'dt_type' => 'DateTime',
                                       '_max_write_batch_size' => 1000,
                                       '_opts' => {
                                                    'port' => '27017',
                                                    'host' => 'ww.xx.yy.zz'
                                                  }
                                     }, 'MongoDB::MongoClient' ),
                 'partial' => 0,
                 '_query' => bless( [
                                      {
                                        'likes' => 0
                                      },
                                      [
                                        'likes'
                                      ],
                                      [
                                        'math'
                                      ],
                                      0
                                    ], 'Tie::IxHash' ),
                 '_tailable' => 0,
                 '_client' => $VAR1->{'_master'},
                 '_limit' => 0,
                 '_agg_batch_size' => 0,
                 '_batch_size' => 0,
                 'slave_okay' => 0,
                 '_request_id' => 0,
                 'immortal' => 0,
                 'started_iterating' => 0
               }, 'MongoDB::Cursor' );
Geek:Eddy
Geek:Stan the Geek

---------------------------------------------------------------
Dropped Database - 'tutorial' = $VAR1 = {
          'ok' => '1',
          'dropped' => 'tutorial'
        };
---------------------------------------------------------------

```
### References:

http://search.cpan.org/dist/MongoDB - Source Code download site for MongDB Perl Driver.

http://www.cpan.org/misc/cpan-faq.html - CPAN Frequently asked questions.

http://docs.mongodb.org/manual/installation - Installation manual for MongDB database.

http://docs.mongodb.org/ecosystem/drivers/perl - Source  For information about the MongoDB Perl Driver.

https://metacpan.org/pod/distribution/MongoDB/lib/MongoDB/Tutorial.pod - Tutorial for MongoDB Perl Driver.