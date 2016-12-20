<!---PACKAGE:Apache Storm--->
<!---DISTRO:SLES 12:1.0.2--->
<!---DISTRO:RHEL 7.1:1.0.2--->

# Apache Storm
The instructions provided below specify the steps to build Apache Storm version 1.0.2 on IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10.
  
_**General Notes:**_   
* _When following the steps below please use a standard permission user unless otherwise specified._
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

##Step 1: Building and installing Apache Storm
####1.1) Install Dependencies

* RHEL 7.1/7.2/7.3
  ```bash
  sudo yum install git java-1.8.0-openjdk-devel.s390x tar  
  ```

* SLES 12-SP1/12-SP2
  ```bash
  sudo zypper install git java-1.8.0-openjdk tar  
  ```

* Ubuntu 16.04/16.10
  ```bash
  sudo apt-get install git tar ant wget openjdk-8-jdk maven curl gpgv nodejs
  ```

####1.2) Install Maven(For RHEL and SLES only)

  Maven v3.0.0 or above is required. Instructions for building Maven can be found [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Maven).
  
####1.3) Install NodeJS SDK (For RHEL and SLES only)

  Install nodejs sdk from  https://developer.ibm.com/node/sdk/ for [**Linux on System z 64-bit**](http://www14.software.ibm.com/cgi-bin/weblap/lap.pl?popup=Y&la_formnum=&li_formnum=L-PIVN-A98LMB&title=IBM%20SDK%20for%20Node.js%20Version%206&accepted_url=http://public.dhe.ibm.com/ibmdl/export/pub/systems/cloud/runtimes/nodejs/6.2.2.0/linux/s390x/ibm-6.2.2.0-node-v6.2.2-linux-s390x.bin).  
  
####1.4) Install rvm and nvm
  ```bash
  curl -L https://get.rvm.io | bash -s stable --autolibs=enabled && source $HOME/.profile  
  command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -  
  wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.26.1/install.sh | bash && source $HOME/.bashrc
  ```
  
####1.5) Set environment variables

* Add the nodejs executable to the path.(For RHEL and SLES only)
  ```bash
  export PATH=$PATH:<Nodejs-Install-Directory>/node/bin  
  ```

* Setup up Maven and JAVA  
  ```bash
  export JAVA_HOME=/usr/lib/jvm/java  #(for RHEL)
  export JAVA_HOME=/usr/lib64/jvm/java  #(for SLES)
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x #(for Ubuntu)
  export M2_HOME=<Your-Maven-location> #(for RHEL and SLES only)
  export PATH=$PATH:$M2_HOME/bin #(for RHEL and SLES only)
  export PATH=$JAVA_HOME/bin:$PATH
  export _JAVA_OPTIONS=-Xmx4096m
  export JVM_ARGS="-Xms4096m -Xmx4096m"
  ```  
  
####1.6) Download the source code

  ```bash
  cd /<source_root>/
  git clone https://github.com/apache/storm.git  
  cd storm  
  git checkout tags/v1.0.2
  ```

####1.7) Modify the `/<source_root>/storm/external/storm-metrics/pom.xml` file

  ```diff
  @@ -33,7 +33,7 @@
   <properties>
     <!-- settings for downloading the sigar native binary complete archive, which is not available in Maven central-->
     <sigar.version>1.6.4</sigar.version>
-    <sigar.download.url>https://magelan.googlecode.com/files/hyperic-sigar-${sigar.version}.zip</sigar.download.url>
+    <sigar.download.url>https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/magelan/hyperic-sigar-${sigar.version}.zip</sigar.download.url>
     <sigar.SHA1>8f79d4039ca3ec6c88039d5897a80a268213e6b7</sigar.SHA1>
     <!-- this will download the sigar ZIP to the local maven repository next to the sigar dependencies,
          so we only download it once -->
  ```
  
####1.8) Buuild Apache Storm

  ```bash
  mvn clean install
  ```
  _**Note:** If you wish to skip the unit tests you can do this by adding -DskipTests to the command line._
  
##Step 2: Testing(Optional)

####2.1) Run unit testcases

  To run Clojure and Java unit tests but no integration tests execute the command
  ```bash
  cd /<source_root>/storm
  mvn test
  ```
  _**Note:** Integration tests require that you activate the profile integration-test and that you specify the maven-failsafe-plugin in the module pom file._
  
##Step 3: Create a Storm distribution (packaging)

  ```bash
  cd /<source_root>/storm/storm-dist/binary 
  mvn package  
  ```
  _**Note:**_ 
  * _After running mvn package you may be asked to enter your GPG/PGP credentials (once for each binary file, in fact). This happens because the packaging step will create *.asc digital signatures for all the binaries, and in the workflow above your GPG private key will be used to create those signatures._
  * _The binaries can be found at `/<source_root>/storm/storm-dist/binary/target/`._

##References:
http://storm.apache.org/releases/current/index.html