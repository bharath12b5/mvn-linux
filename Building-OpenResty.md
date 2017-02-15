<!---PACKAGE:OpenResty--->
<!---DISTRO:SLES 12.x:1.9.15.1--->
<!---DISTRO:RHEL 7.x:1.9.15.1--->
<!---DISTRO:Ubuntu 16.x:1.9.15.1--->

# Building OpenResty 

The instructions provided below specify the steps to build OpenResty 1.9.15.1 on IBM z Systems for following distibutions:
* RHEL (7.1, 7.2, 7.3)
* SLES (12, 12 SP1, 12 SP2) 
* Ubuntu (16.04, 16.10) 

### Prerequisites:
  * LuaJIT 
  -- Instructions for building LuaJIT can be found [here]( https://github.com/linux-on-ibm-z/docs/wiki/Building-LuaJIT)

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory /\<source_root\>/ will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._


### Step 1: Install the dependencies
   
   * RHEL (7.1, 7.2, 7.3)
   
    ```
	sudo yum install -y git tar wget make gcc gcc-c++ unix2dos hg cpan perl postgresql-devel unix2dos patch pcre-devel readline-devel openssl openssl-devel 
    ```
    
   * SLES (12, 12 SP1, 12 SP2)
   
    ```
	sudo zypper install -y git tar wget make gcc gcc-c++ unix2dos hg perl postgresql-devel unix2dos patch pcre-devel readline-devel openssl openssl-devel aaa_base
    ```
    
   * Ubuntu (16.04, 16.10)
   
    ```
    sudo apt-get update
	sudo apt-get install -y git tar wget make gcc dos2unix hgview libreadline-dev patch libpcre3-dev libpcre3 libcurl4-openssl-dev libncursesada*-dev postgresql libpq-dev
    sudo ln -s make /usr/bin/gmake
    ```

### Step 2: Set environment variables
  ```bash
  export PATH=$PATH:/sbin
  ```
    
### Step 3: Download the source code
  ```bash
  cd /<source_root>
  git clone https://github.com/openresty/openresty.git
  cd openresty
  git checkout v1.9.15.1
  ```

### Step 4: Build and install OpenResty 
  ```bash    
  make	
  rm -rf /<source_root>/openresty/openresty-1.9.15.1/bundle/LuaJIT-2.1-20160517/
  cp -r /<source_root>/LuaJIT /<source_root>/openresty/openresty-1.9.15.1/bundle/LuaJIT-2.1-20160517/
  cd /<source_root>/openresty/openresty-1.9.15.1
  ./configure --without-http_redis2_module --with-http_iconv_module --with-http_postgres_module  -j2 
  make -j2 
  sudo make install
  ```
    
### Step 5: Configure Nginx module
  ```bash
  cd /<source_root>/openresty/openresty-1.9.15.1/build/nginx-1.9.15
  ./configure && make && sudo make install
  ```

### Step 6: Install cpan modules
  ```bash
  sudo cpan Cwd IPC::Run3 Test::Base Test::LongString
  ```
	  
_Note: For options prompted please select the default option._	

### Step 7: Edit file `/<source_root>/openresty/t/sanity.t`

 * Ubuntu (16.04, 16.10)

  ```diff
  @@ -1807,8 +1807,8 @@ clean:
  platform: linux (linux)
  cp -rp bundle/ build
  cd build
- export LIBPQ_LIB='/usr/lib64'
- export LIBPQ_INC='/usr/include'
+ export LIBPQ_LIB='/usr/lib/s390x-linux-gnu'
+ export LIBPQ_INC='/usr/include/postgresql'
  cd lua-5.1.5
  gmake linux
  gmake install INSTALL_TOP=$OPENRESTY_BUILD_DIR/lua-root//usr/local/openresty/lua/
@@ -1836,7 +1836,7 @@ sh ./configure --prefix=/usr/local/openresty/nginx \
       --add-module=../redis-nginx-module-0.3.7 \
       --add-module=../rds-json-nginx-module-0.14 \
       --add-module=../rds-csv-nginx-module-0.07 \
-      --with-ld-opt='-Wl,-rpath,/usr/lib64' \
+      --with-ld-opt='-Wl,-rpath,/usr/lib/s390x-linux-gnu' \
       --with-http_ssl_module
 cd ../..
 Type the following commands to build and install:
 
   ```

 * SLES (12, 12 SP1, 12 SP2)
 
  ```diff
  
  @@ -1807,8 +1807,8 @@ clean:
  platform: linux (linux)
  cp -rp bundle/ build
  cd build
- export LIBPQ_LIB='/usr/lib64'
- export LIBPQ_INC='/usr/include'
+ export LIBPQ_LIB='/usr/lib/postgresql94/lib64'
+ export LIBPQ_INC='/usr/include/pgsql'
  cd lua-5.1.5
  gmake linux
  gmake install INSTALL_TOP=$OPENRESTY_BUILD_DIR/lua-root//usr/local/openresty/lua/
  @@ -1836,7 +1836,7 @@ sh ./configure --prefix=/usr/local/openresty/nginx \
       --add-module=../redis-nginx-module-0.3.7 \
       --add-module=../rds-json-nginx-module-0.14 \
       --add-module=../rds-csv-nginx-module-0.07 \
-      --with-ld-opt='-Wl,-rpath,/usr/lib64' \
+      --with-ld-opt='-Wl,-rpath,/usr/lib/postgresql94/lib64' \
       --with-http_ssl_module
  cd ../..
  Type the following commands to build and install:

 ```

### Step 8: Run test cases (Optional)
  ```bash
  cd /<source_root>/openresty
  make test
  ```

### References
https://openresty.org/
