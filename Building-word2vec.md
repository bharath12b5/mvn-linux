<!---PACKAGE:word2vec--->
<!---DISTRO:SLES 12:0.1c--->
<!---DISTRO:SLES 11:0.1c--->
<!---DISTRO:RHEL 7.1:0.1c--->
<!---DISTRO:RHEL 6.6:0.1c--->
<!---DISTRO:Ubuntu 16.x:0.1c--->

# Building word2vec

The instructions provided below specify the steps to build word2vec v0.1c on Linux on z Systems for RHEL 6.6/7.2, SLES11/12 and Ubuntu 16.04.

##### General Notes:
      
i) When following the steps below please use a standard permission user unless otherwise specified.

ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it.

### Building word2vec

1. Install standard utilities, packages and platform specific dependencies

    RHEL7.2/RHEL6.6:

    ```
    sudo yum install -y git gcc make wget tar unzip
     ```

    SLES11/SLES12 (SP1):
    ```
    sudo zypper install -y git gcc make wget tar unzip
    ```

    Ubuntu 16.04
    ```
    sudo apt-get update
    sudo apt-get install -y git gcc make wget tar unzip
    ```
 
2. Create a working directory and download word2vec source code 

   ```
    mkdir /<source_root>/
    cd /<source_root>/
    wget https://storage.googleapis.com/google-code-archive-source/v2/code.google.com/word2vec/source-archive.zip
    unzip source-archive.zip
   ```

3. Build word2vec 

   ```
    cd word2vec/trunk
    make CFLAGS="-lm -pthread -O3 -Wall -funroll-loops"
   ```

4. Set environment variables 

   ```	
    export PATH=$PATH:/<source_root>/word2vec/trunk
   ```

5. Test word2vec using demo scripts 
    
   ```
    ./demo-word.sh
    ./demo-phrases.sh
   ```
    _**Note:**_ Enter test corpus as input and get word vectors as output, e.g. Input=france
6. Run word2vec binary

    
   ```
    word2vec
   ```  
    _**Note:**_ The word2vec tool takes a text corpus as input and produces the word vectors as output.

### References:
https://code.google.com/archive/p/word2vec/