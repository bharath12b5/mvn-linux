# Building Cloud Foundry Ruby Buildpack

### _**General Note:**_
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

This is an instruction to build Cloud Foundry Ruby Buildpack.

### Step 1 : Get the source code
```
cd /<source_root>
git clone https://github.com/cloudfoundry/ruby-buildpack.git
```

### Step 2 : Build necessary dependencies
All the dependencies are listed in `manifest.yml`, and the following dependencies need to be changed to be s390x compatible:

#### 1. OpenJDK
```
sudo apt-get install openjdk-8-jdk
cd /<source_root>/ruby-buildpack
mkdir tmp
cp -rL /usr/lib/jvm/java-8-openjdk-s390x/* ./tmp
```

Then compress this directory:
```
cd /<source_root>/ruby-buildpack/tmp
tar czf ../openjdk-1.8.0.tar.gz ./*
```

#### 2. Ruby
##### 1. [Build warden root filesystems](https://github.com/linux-on-ibm-z/docs/wiki/Building-Cloud-Foundry#building-warden-root-filesystems)
This root filesystems will be used in Cloud Foundry binary-builder.

##### 2. Prepare Cloud Foundry binary-builder
Cloud Foundry binary-builder is a tool to build the binary dependencies for Cloud Foundry buildpacks. However, it doesn't cover all dependencies of all supported buildpacks at current moment. First, get the source code:
```
cd /<source_root>
git clone https://github.com/cloudfoundry/binary-builder.git
cd binary-builder
```

Make changes to the following files:
* `/<source_root>/recipe/base.rb`
```diff
@@ -27,7 +27,7 @@ class BaseRecipe < MiniPortile
   end

   def archive_filename
-    "#{name}-#{version}-linux-x64.tgz"
+    "#{name}-#{version}-linux-s390x.tgz"
   end

   def archive_files
```

##### 3. Binary-builder changes for Ruby dependencies
To build the Ruby binary, the Ruby source code package will be downloaded from `https://cache.ruby-lang.org/pub/ruby`. Binary-builder first downloads it and checks its MD5 checksum for authenticity and integrity. The checksum is easy to get, for example: 
```
wget https://cache.ruby-lang.org/pub/ruby/ruby-2.3.1.tar.gz
md5sum ruby-2.3.1.tar.gz
```

After obtaining the MD5 checkshum, issue command to build the Ruby binary:
```
cd /<source_root>/binary-builder
docker run -w /binary-builder -v `pwd`:/binary-builder -it cloudfoundry/cflinuxfs2 ./bin/binary-builder --name=ruby --version=2.3.1 --md5=0d896c2e7fd54f722b399f407e48a4c6
```
Then move the generated tgz file to the ruby-buildpack directory.

#### 3. JRuby
Install JRuby,
```
sudo apt-get install jruby
rm /<source_root>/ruby-buildpack/tmp/* -rf
cp -rL /usr/share/jruby/* /<source_root>/ruby-buildpack/tmp
```

Then, compress the JRuby dependency:
```
cd /<source_root>/ruby-buildpack
tar xzf jruby-9.1.2.0_ruby-2.3.1-linux-s390x.tgz ./tmp/*
```

### Step 3 : Change files before building ruby-buildpack
```
cd /<source_root>/ruby-buildpack
```

Change the following files:
* `/<source_root>/ruby-buildpack/lib/cloud_foundry/language_pack/helpers/node_installer.rb`
```diff
@@ -11,5 +11,5 @@ class LanguagePack::NodeInstaller

   const = 'MODERN_BINARY_PATH'
   self.send(:remove_const, const) if self.const_defined?(const)
-  MODERN_BINARY_PATH = "node-v#{MODERN_NODE_VERSION}-linux-x64"
+  MODERN_BINARY_PATH = "node-v#{MODERN_NODE_VERSION}-linux-s390x"
 end

```

* `/<source_root>/ruby-buildpack/lib/language_pack/helpers/node_installer.rb`
```diff
@@ -1,6 +1,6 @@
 class LanguagePack::NodeInstaller
   MODERN_NODE_VERSION = "TO_BE_REPLACED_BY_CF_DEFAULTS"
-  MODERN_BINARY_PATH  = "node-v#{MODERN_NODE_VERSION}-linux-x64"
+  MODERN_BINARY_PATH  = "node-v#{MODERN_NODE_VERSION}-linux-s390x"

   LEGACY_NODE_VERSION = "0.4.7"
   LEGACY_BINARY_PATH = "node-#{LEGACY_NODE_VERSION}"
```

* `/<source_root>/ruby-buildpack/manifest.yml`
```diff
@@ -4,7 +4,7 @@ default_versions:
 - name: ruby
   version: 2.3.1
 - name: node
-  version: 4.5.0
+  version: 6.4.0
 url_to_dependency_map:
 - match: ruby-(.*?)-jruby-(.*?)\.tgz
   name: jruby
@@ -26,10 +26,10 @@ url_to_dependency_map:
   version: 0
 - match: openjdk1.8-latest
   name: openjdk1.8-latest
-  version: 1.8.0_101
+  version: 1.8.0
 - match: node
   name: node
-  version: 4.5.0
+  version: 6.4.0
 dependencies:
 - name: libyaml
   version: 0.1.6
@@ -56,69 +56,27 @@ dependencies:
   cf_stacks:
   - cflinuxfs2
 - name: openjdk1.8-latest
-  version: 1.8.0_101
-  uri: https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_101.tar.gz
-  md5: 66281ba0c6d42ff3a5b1770e1aa8369e
-  cf_stacks:
-  - cflinuxfs2
-- name: ruby
-  version: 2.3.0
-  md5: 535342030a11abeb11497824bf642bf2
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/ruby/ruby-2.3.0-linux-x64.tgz
+  version: 1.8.0
+  uri: file:///path/ruby-buildpack/openjdk-1.8.0.tar.gz
+  md5: 2a2a4e9ebaa1de219098a8a0f2362695
   cf_stacks:
   - cflinuxfs2
 - name: ruby
   version: 2.3.1
-  md5: c55c51d66a18123363e7f96635b54717
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/ruby/ruby-2.3.1-linux-x64.tgz
-  cf_stacks:
-  - cflinuxfs2
-- name: ruby
-  version: 2.2.4
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/ruby/ruby-2.2.4-linux-x64.tgz
-  md5: 10f417882bf68dca0f3047e6fe7426f7
-  cf_stacks:
-  - cflinuxfs2
-- name: ruby
-  version: 2.2.5
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/ruby/ruby-2.2.5-linux-x64.tgz
-  md5: faf863c214dd9972f18daf9861fc3898
-  cf_stacks:
-  - cflinuxfs2
-- name: ruby
-  version: 2.1.8
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/ruby/ruby-2.1.8-linux-x64.tgz
-  md5: 6610af292a8b5666858f1566c3349108
-  cf_stacks:
-  - cflinuxfs2
-- name: ruby
-  version: 2.1.9
-  md5: 095249c4ed574b135b2aeb699c1c8d42
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/ruby/ruby-2.1.9-linux-x64.tgz
-  cf_stacks:
-  - cflinuxfs2
-- name: jruby
-  version: ruby-1.9.3-jruby-1.7.25
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/jruby/jruby-1.7.25_ruby-1.9.3-linux-x64.tgz
-  md5: 25f06b19a537221e02b88f30207f1afc
-  cf_stacks:
-  - cflinuxfs2
-- name: jruby
-  version: ruby-2.0.0-jruby-1.7.25
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/jruby/jruby-1.7.25_ruby-2.0.0-linux-x64.tgz
-  md5: ff0cf651f8521ad4a70a673c8ed99760
+  md5: a879d6ed4ce5e08e195875a8fe4e29a0
+  uri: file:///<source_root>/ruby-buildpack/ruby-2.3.1-linux-s390x.tgz
   cf_stacks:
   - cflinuxfs2
 - name: jruby
   version: ruby-2.3.0-jruby-9.1.2.0
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/jruby/jruby-9.1.2.0_ruby-2.3.0-linux-x64.tgz
-  md5: 1aa7047b7f7c233130ffef1af71bf7de
+  uri: file:///<source_root>/ruby-buildpack/jruby-9.1.2.0_ruby-2.3.1-linux-s390x.tgz
+  md5: c3fc8fe5137e88ee76cc4755694e9f59
   cf_stacks:
   - cflinuxfs2
 - name: node
-  version: 4.5.0
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/node/node-4.5.0-linux-x64.tgz
-  md5: a0784a91f4cc412efa1472e13acc500a
+  version: 6.4.0
+  uri: file:///<source_root>/ruby-buildpack/node-6.4.0-linux-s390x.tgz
+  md5: 57f28f5bc5ca0654ff2df719a3d211a4
   cf_stacks:
   - cflinuxfs2
 exclude_files:
```

Here in `manifest.yml`, you need to carefully replace the dependencies which we built earlier before, and change the URI and MD5. The URI is the location of the dependencies and the MD5 value can be obtained by issuing an md5sum command as mentioned before. Some dependencies can have different versions, e.g., Ruby, and you can simply remove their entries in the manifest if you don't need them in your buildpack. Or if you want them, you can build their binaries as mentioned before but with their corresponding versions. In our example, we only keep one version for each dependencies.
 
### Step 4 : Build Ruby Builpack
After all the changes, build the buildpack:
```
git submodule update --init
BUNDLE_GEMFILE=cf.Gemfile bundle
BUNDLE_GEMFILE=cf.Gemfile bundle exec buildpack-packager --cached
```

The dependency cached buildpack will be built as a zip file. To use it in your cloud foundry, after login to your Cloud Foundry instance, 
```
cf create-buildpack custom_ruby_buildpack ruby_buildpack-cached-custom.zip 1
cd /path_to_ruby_app
cf push my_app -b custom_ruby_buildpack
```