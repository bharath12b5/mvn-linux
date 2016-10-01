# Building Cloud Foundry Node.js Buildpack

### _**General Note:**_
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

This is an instruction to build Cloud Foundry Node.js Buildpack.

### Step 1 : Get the source code
```
cd /<source_root>
git clone https://github.com/cloudfoundry/nodejs-buildpack.git
```

### Step 2 : Build necessary dependencies

#### 1. Node.js for s390x

Download IBM Node.js from `https://developer.ibm.com/node/sdk/` and install Node.js
```
chmod +x ibm-6.4.0.0-node-v6.4.0-linux-s390x.bin
./ibm-6.4.0.0-node-v6.4.0-linux-s390x.bin
```

Follow the default, then Node.js will be installed in `/root/ibm/node`.
```
cd /root/ibm/node/bin
ln -sf ../lib/node_modules/npm/bin/npm-cli.js npm
ln -sf ../lib/node_modules/appmetrics/bin/appmetrics-cli.js node-hc
```

Prepare the Node.js dependency, 
```
cd /<source_root>/nodejs-buildpack
mkdir tmp
cp -rL /root/ibm/node ./tmp
cd tmp
tar czf ../node-6.4.0-linux-s390x.tgz ./node
```

### Step 3 : Make changes to the following files:

* node binary
```
cd /<source_root>/nodejs-buildpack
cp /root/ibm/node/bin/node ./bin
```

* jq binary
```
sudo apt-get install jq
cp /usr/bin/jq ./vendor/jq-linux
```

* `/<source_root>/nodejs-buildpack/lib/binaries.sh`
```diff
@@ -24,12 +24,9 @@ install_nodejs() {
   else
     echo "Downloading and installing node $resolved_version..."
   fi
-  local download_url="https://s3pository.heroku.com/node/v$resolved_version/node-v$resolved_version-$os-$cpu.tar.gz"
-  curl "`translate_dependency_url $download_url`" --silent --fail --retry 5 --retry-max-time 15 -o /tmp/node.tar.gz || (>&2 $BP_DIR/compile-extensions/bin/recommend_dependency $download_url && false)
-  echo "Downloaded [`translate_dependency_url $download_url`]"
-  tar xzf /tmp/node.tar.gz -C /tmp
+  tar xzf $BP_DIR/node-6.4.0-linux-s390x.tgz -C /tmp
   rm -rf $dir/*
-  mv /tmp/node-v$resolved_version-$os-$cpu/* $dir
+  mv /tmp/node/* $dir
   chmod +x $dir/bin/*
 }
```

* `/<source_root>/nodejs-buildpack/lib/json.sh`
```diff
@@ -6,7 +6,7 @@ get_cpu() {
   if [[ "$(uname -p)" = "i686" ]]; then
     echo "x86"
   else
-    echo "x64"
+    echo "s390x"
   fi
 }
```

* `/<source_root>/nodejs-buildpack/manifest.yml`
```diff
@@ -16,62 +16,8 @@ url_to_dependency_map:
   version: "$1"
 dependencies:
 - name: node
-  version: 0.10.45
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-0.10.45-linux-x64.tgz
-  md5: b6607379e8cdcfa3763acc12fe40cef9
-  cf_stacks:
-  - cflinuxfs2
-- name: node
-  version: 0.10.46
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-0.10.46-linux-x64.tgz
-  md5: 2e02c3350a0b81e8b501ef3ea637a93b
-  cf_stacks:
-  - cflinuxfs2
-- name: node
-  version: 0.12.14
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-0.12.14-linux-x64.tgz
-  md5: 510555de82cc985898731dc7c18a6dfb
-  cf_stacks:
-  - cflinuxfs2
-- name: node
-  version: 0.12.15
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-0.12.15-linux-x64.tgz
-  md5: 317b688dde627bb85e7d575870f08eb2
-  cf_stacks:
-  - cflinuxfs2
-- name: node
-  version: 4.4.6
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-4.4.6-linux-x64.tgz
-  md5: 33822ae3f92ac9586d73dee3c42a4bf2
-  cf_stacks:
-  - cflinuxfs2
-- name: node
-  version: 5.11.1
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-5.11.1-linux-x64.tgz
-  md5: c6da910f661470d01e7920a1d3efaee2
-  cf_stacks:
-  - cflinuxfs2
-- name: node
-  version: 5.12.0
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-5.12.0-linux-x64.tgz
-  md5: 006d5be71aa68c7cccdd5c2c9f1d0fc0
-  cf_stacks:
-  - cflinuxfs2
-- name: node
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-4.4.7-linux-x64.tgz
-  md5: 81fb9f3b687bdb931e9f06fa3b7c68fe
-  cf_stacks:
-  - cflinuxfs2
-- name: node
-  version: 6.3.0
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-6.3.0-linux-x64.tgz
-  md5: 5992b136e75fa0466f38ce7903ee5d8e
-  cf_stacks:
-  - cflinuxfs2
-- name: node
-  version: 6.3.1
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-6.3.1-linux-x64.tgz
-  md5: e0349642f1c8556913ef95b16e737a64
+  version: 6.4.0
+  uri: file:///<source_root>/nodejs-buildpack/node-6.4.0-linux-s390x.tgz
+  md5: a2dc27af061fc887d7d8f8c78d3e7878
   cf_stacks:
   - cflinuxfs2
```
Here in `manifest.yml`, you need to carefully replace the dependencies which we built before, and change the URI and MD5. The URI is the location of the dependencies and the MD5 value can be obtained by issuing an md5sum command. Some dependencies can have different versions, and you can simply remove their entries in the manifest if you don't need them in your buildpack. Or if you want them, you can build their binaries but with their corresponding versions. In our example, we only keep one version of each dependencies.

### Step 4 : Build Node.js Buildpack

After all the changes, build the buildpack:
```
git submodule update --init
BUNDLE_GEMFILE=cf.Gemfile bundle
BUNDLE_GEMFILE=cf.Gemfile bundle exec buildpack-packager --cached
```

The dependency cached buildpack will be built as a zip file. To use it in your Cloud Foundry, after login to your Cloud Foundry instance, 
```
cf create-buildpack custom_nodejs_buildpack nodejs_buildpack-cached-custom.zip 1
cd /path_to_nodejs_app
cf push nodejs -b custom_nodejs_buildpack
```
