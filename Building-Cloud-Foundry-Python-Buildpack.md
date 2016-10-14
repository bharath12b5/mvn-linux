# Building Cloud Foundry Python Buildpack

### _**General Note:**_
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

This is an instruction to build Cloud Foundry Python Buildpack. For proof of concept purpose, this instruction builds buildpack supporting Python 3.5. If Python 2.7 support is needed, you can contact us. Miniconda support is removed from this buildpack since there is not a Conda repository available right now for Linux on z systems.

### Step 1 : Get the source code
```
cd /<source_root>
git clone https://github.com/cloudfoundry/python-buildpack.git
```

### Step 2 : Build necessary dependencies

#### 1. [Build warden root filesystems](https://github.com/linux-on-ibm-z/docs/wiki/Building-Cloud-Foundry#building-warden-root-filesystems)

#### 2. [Prepare Cloud Foundry Binary-Builder](https://github.com/linux-on-ibm-z/docs/wiki/Building-Cloud-Foundry-Ruby-Buildpack#2-prepare-cloud-foundry-binary-builder)

#### 3. Python 3.5

We use Cloud Foundry binary-builder to build Python 3.5.
```
cd /<source_root>/binary-builder
```

Run the binary-builder to build the binary:
```
docker run -w /binary-builder -v `pwd`:/binary-builder -it cloudfoundry/cflinuxfs2 ./bin/binary-builder --name=python --version=3.5.2 --md5=3fe8434643a78630c61c6464fe2e7e72
cp python-3.5.2-linux-s390x.tgz /<source_root>/python-buildpack
```
    
Note the binary-builder will download the source code from `https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tgz`, so the MD5 hash code is provided in the command to provide authenticity and integrity of the downloaded package.

#### 4. libmemcache 1.0.18

Build libmemcache from the source code:
```
mkdir -p /<source_root>/tmp/libmemcached-tmp
cd /<source_root>/tmp
wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
tar xzvf libmemcached-1.0.18
cd libmemcached-1.0.18
./bootstrap.sh autoreconf
./configure --prefix=/<source_root>/tmp/libmemcached-tmp
make
make install
cd /<source_root>/tmp/libmemcached-tmp
tar czf /<source_root>/python-buildpack/libmemcache.tar.gz ./*
```

#### 5. libffi 3.1

Build libffi from the source code:
```
mkdir -p /<source_root>/tmp/libffi-tmp
cd /<source_root>/tmp
git clone https://github.com/libffi/libffi.git
cd libffi
git checkout v3.1
./autogen.sh
./configure --prefix=/<source_root>/tmp/libffi-tmp
make
make install
```

Make following changes

* `/<source_root>/tmp/libffi-tmp/lib/libffi.la`
```diff
@@ -41 +41 @@
-libdir='/<source_root>/tmp/libffi-tmp/lib/../lib'
+libdir='/app/.heroku/vendor/lib/../lib'
```

* `/<source_root>/tmp/lib/pkgconfig/libffi.pc`
```diff
@@ -1 +1 @@
-prefix=/<source_root>/tmp/libffi-tmp
+prefix=/app/.heroku/vendor
```

Then pack the directory:
```
cd /<source_root>/tmp/libffi-tmp
tar czf /<source_root>/python-buildpack/libffi.tar.gz ./*
```

### Step 3 : Change the following files:

* `/<source_root>/python-buildpack/bin/compile`
```diff
@@ -28,11 +28,11 @@ CACHE_DIR=$2
 ENV_DIR=$3

 # Use miniconda if environment.yml exists and exit
-if [ -f $BUILD_DIR/environment.yml ]; then
-  echo "----------------- USING CONDA BUILDPACK -----------------"
-  $BIN_DIR/steps/conda-install $BUILD_DIR $CACHE_DIR
-  exit 0
-fi
+#if [ -f $BUILD_DIR/environment.yml ]; then
+#  echo "----------------- USING CONDA BUILDPACK -----------------"
+#  $BIN_DIR/steps/conda-install $BUILD_DIR $CACHE_DIR
+#  exit 0
+#fi

 $ROOT_DIR/compile-extensions/bin/check_stack_support

@@ -231,10 +231,10 @@ source $BIN_DIR/steps/pylibmc
 source $BIN_DIR/steps/cryptography

 # Support for Geo libraries.
-sub-env $BIN_DIR/steps/geo-libs
+#sub-env $BIN_DIR/steps/geo-libs

 # GDAL support.
-source $BIN_DIR/steps/gdal
+#source $BIN_DIR/steps/gdal

 # Install dependencies with Pip.
 source $BIN_DIR/steps/pip-install
```

* `/<source_root>/python-buildpack/bin/step/python`
```diff
@@ -31,11 +31,13 @@ if [ ! "$SKIP_INSTALL" ]; then
     url="https://lang-python.s3.amazonaws.com/$STACK/runtimes/$PYTHON_VERSION.tar.gz"
     translated_url=$(translate_dependency_url $url) || exit_code=$?
     filtered_url=$(filter_dependency_url $translated_url) || true
-    if [[ $exit_code != 0 ]] ; then
-      $ROOT_DIR/compile-extensions/bin/recommend_dependency $url
-      exit 1
-    fi
-    curl "$translated_url" -s | tar zxv -C .heroku/python &> /dev/null
+#    if [[ $exit_code != 0 ]] ; then
+#      $ROOT_DIR/compile-extensions/bin/recommend_dependency $url
+#      exit 1
+#    fi
+#    curl "$translated_url" -s | tar zxv -C .heroku/python &> /dev/null
+
+    tar zxvf $ROOT_DIR/python-3.5.2-linux-s390x.tgz -C .heroku/python > /dev/null
     echo "Downloaded [$filtered_url]"

   bpwatch stop install_python
```

* `/<source_root>/python-buildpack/manifest.yml`
```diff
@@ -2,7 +2,7 @@
 language: python
 default_versions:
   - name: python
-    version: 2.7.12
+    version: 3.5.2
 url_to_dependency_map:
   - match: python-(\d+\.\d+\.\d+)
     name: python
@@ -19,62 +19,20 @@ dependencies:
     version: 1.0.18
     cf_stacks:
       - cflinuxfs2
-    uri: https://buildpacks.cloudfoundry.org/python/binaries/cflinuxfs2/libmemcache.tar.gz
-    md5: 6b40600ab7c5bd52b5c6eedd18efb651
+    uri: file:///localbox/cf_local/python-buildpack/libmemcache.tar.gz
+    md5: 8524b2ba8d673d4f6b00e5002a4433f2
   - name: libffi
     version: "3.1"
     cf_stacks:
       - cflinuxfs2
-    uri: https://buildpacks.cloudfoundry.org/python/binaries/cflinuxfs2/libffi.tar.gz
-    md5: 83028a299b8ac323bda8a56c5c70d4cd
-  - name: python
-    version: 2.7.11
-    cf_stacks:
-      - cflinuxfs2
-    uri: https://buildpacks.cloudfoundry.org/concourse-binaries/python/python-2.7.11-linux-x64.tgz
-    md5: 6a93c298ce97d4eb6b3ec7039f9ae439
-  - name: python
-    version: 2.7.12
-    cf_stacks:
-      - cflinuxfs2
-    uri: https://buildpacks.cloudfoundry.org/concourse-binaries/python/python-2.7.12-linux-x64.tgz
-    md5: 5b49c4c1ba9ceae55f5d4bd0b95863f4
-  - name: python
-    version: 3.3.5
-    cf_stacks:
-      - cflinuxfs2
-    uri: https://buildpacks.cloudfoundry.org/concourse-binaries/python/python-3.3.5-linux-x64.tgz
-    md5: f32e11f2d039dae0d6574403a80b485d
-  - name: python
-    version: 3.3.6
-    cf_stacks:
-      - cflinuxfs2
-    uri: https://buildpacks.cloudfoundry.org/concourse-binaries/python/python-3.3.6-linux-x64.tgz
-    md5: 08dee09d32477c7f0497e736c0c7967b
-  - name: python
-    version: 3.4.4
-    cf_stacks:
-      - cflinuxfs2
-    uri: https://buildpacks.cloudfoundry.org/concourse-binaries/python/python-3.4.4-linux-x64.tgz
-    md5: eeeaf592c843fbc05528d782f20486fc
-  - name: python
-    version: 3.4.5
-    cf_stacks:
-      - cflinuxfs2
-    uri: https://buildpacks.cloudfoundry.org/concourse-binaries/python/python-3.4.5-linux-x64.tgz
-    md5: 119671dbbb94170e69da5f3247f6ee6e
-  - name: python
-    version: 3.5.1
-    cf_stacks:
-      - cflinuxfs2
-    uri: https://buildpacks.cloudfoundry.org/concourse-binaries/python/python-3.5.1-linux-x64.tgz
-    md5: a292283d7eec49d7ea5c60036f633740
+    uri: file:///localbox/cf_local/python-buildpack/libffi.tar.gz
+    md5: a5c7d38d35a7e5997ea6b17e69fcd43e
   - name: python
     version: 3.5.2
     cf_stacks:
       - cflinuxfs2
-    uri: https://buildpacks.cloudfoundry.org/concourse-binaries/python/python-3.5.2-linux-x64.tgz
-    md5: 701a135b3228075c2ce59c527bc92e1e
+    uri: file:///localbox/cf_local/python-buildpack/python-3.5.2-linux-s390x.tgz
+    md5: caa11677298c8994ac3dcb582a7f2736
 exclude_files:
   - .git/
   - .gitignore
```
Here in `manifest.yml`, you need to carefully replace the dependencies which we built earlier before, and change the URI and MD5. The URI is the location of the dependencies and the MD5 value can be obtained by issuing an md5sum command as mentioned before. Some dependencies can have different versions, and you can simply remove their entries in the manifest if you don't need them in your buildpack. Or if you want them, you can build their binaries like mentioned before but with their corresponding versions. In our example, we only keep one version of each dependencies.

* `/<source_root>/python-buildpack/vendor/bpwatch/bpwatch.zip`
```
cd /<source_root>/python-buildpack/vendor/bpwatch
mkdir bpwatch-tmp
unzip bpwatch.zip -d ./bpwatch-tmp
```
Change `/<source_root>/python-buildpack/vendor/bpwatch/bpwatch-tmp/bp_cli.py`
```diff
@@ -37 +37 @@
-        print get_state()
+        print (get_state())
```
And rezip it to replace the original `bpwatch.zip`:
```
zip -r bpwatch.zip ./bpwatch-tmp/*
rm -rf ./bpwatch-tmp
```

### Step 4 : Build Python Builpack
After all the changes, build the buildpack:
```
git submodule update --init
BUNDLE_GEMFILE=cf.Gemfile bundle
BUNDLE_GEMFILE=cf.Gemfile bundle exec buildpack-packager --cached
```
The dependency cached buildpack will be built as a zip file. To use it in your cloud foundry, after login to your cloud foundry instance, 
```
cf create-buildpack custom_python_buildpack python_buildpack-cached-custom.zip 1
cd /path_to_python_app
cf push python_app -b custom_python_buildpack
```
