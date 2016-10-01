# Building Cloud Foundry Go Buildpack

### _**General Note:**_
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

This is an instruction to build Cloud Foundry Go Buildpack.

### Step 1 : Get the source code
```
cd /<source_root>
git clone https://github.com/cloudfoundry/go-buildpack.git
```

### Step 2 : Build necessary dependencies

#### 1. [Build warden root filesystems](https://github.com/rishigits/MyFirstRepository/wiki#building-warden-root-filesystems)

#### 2. [Prepare Cloud Foundry Binary-Builder](https://github.com/rishigits/MyFirstRepository/wiki/Building-Cloud-Foundry-Ruby-Buildpack#2-preparing-cloud-foundry-binary-builder)

#### 3. Glide

We use Cloud Foundry binary-builder to build Glide.
```
cd /<source_root>/binary-builder
```

Make changes to the following files:

* `/<source_root>/binary-builder/recipe/glide.rb`

```diff
@@ -10,10 +10,8 @@ class GlideRecipe < BaseRecipe

     # Installs go 1.6.2 binary to /usr/local/go/bin
     Dir.chdir("/usr/local") do
-      go_download = "https://storage.googleapis.com/golang/go1.6.2.linux-amd64.tar.gz"
-      go_tar = "go.tar.gz"
-      system("curl -L #{go_download} -o #{go_tar}")
-      system("tar xf #{go_tar}")
+      system("apt-get -y install golang")
+      system("cp -rL /usr/lib/go-1.6 ./go")
     end

     FileUtils.rm_rf("#{tmp_path}/glide")
```

Run the binary-builder to build the binary:
```
cd /<source_root>/binary-builder
docker run -w /binary-builder -v `pwd`:/binary-builder -it cloudfoundry/cflinuxfs2 ./bin/binary-builder --name=glide --version=v0.11.1 --sha256=3c4958d1ab9446e3d7b2dc280cd43b84c588d50eb692487bcda950d02b9acc4c
cp glide-v0.11.1-linux-s390x.tgz /<source_root>/go-buildpack
```
    
Note the binary-builder will download the source code from `https://github.com/Masterminds/glide/archive/v0.11.1.tar.gz`, so the SHA-256 hash code is provided in the command to provide authenticity and integrity of the downloaded package.

#### 2. Godep

Change the following files:

* `/<source_root>/binary-builder/recipe/godep.rb`
```diff
@@ -10,10 +10,8 @@ class GodepMeal < BaseRecipe

     # Installs go 1.6.2 binary to /usr/local/go/bin
     Dir.chdir("/usr/local") do
-      go_download = "https://storage.googleapis.com/golang/go1.6.2.linux-amd64.tar.gz"
-      go_tar = "go.tar.gz"
-      system("curl -L #{go_download} -o #{go_tar}")
-      system("tar xf #{go_tar}")
+      system("apt-get -y install golang")
+      system("cp -rL /usr/lib/go-1.6 ./go")
     end

     FileUtils.rm_rf("#{tmp_path}/godep")
```

Run the binary-builder to build the binary:
```
cd /<source_root>/binary-builder
docker run -w /binary-builder -v `pwd`:/binary-builder -it cloudfoundry/cflinuxfs2 ./bin/binary-builder --name=godep --version=v74 --sha256=e68c7766c06c59327a4189fb929d390e1cc7a0c4910e33cada54cf40f40ca546
cp godep-v14-linux-s390x.tgz /<source_root>/go-buildpack
```
The binary-builder will download the source code from `https://github.com/tools/godep/archive/v74.tar.gz`, so the SHA-256 hash code is provided in the command to provide authenticity and integrity of the downloaded package.

#### 3. Golang

Install Golang,
```
apt-get install -y golang
```

Golang is actually installed in `/usr/lib/go-1.6`, create a copy of this directory:
```
cd /<source_root>/go-buildpack
mkdir tmp
cp -rL /usr/lib/go-1.6 ./tmp/go
```

Then compress this Go directory:
```
cd /<source_root>/go-buildpack/tmp
tar xzf ../go1.6.linux-s390x.tar.gz ./go
```
Note that the top directory after decompression of the zip file should be "go".

### Step 3 : Change the following files:

* jq binary
```
cd /<source_root>/go-buildpack
mkdir -p ./linux-s390x/bin
sudo apt-get install jq
cp /usr/bin/jq ./linux-s390x/bin
```

* `/<source_root>/go-buildpack/bin/compile`
```diff
@@ -197,7 +197,7 @@ else
 fi

 ver=$(expandVer $buildpack $ver)
-file=${GOFILE:-$ver.linux-amd64.tar.gz}
+file=${GOFILE:-$ver.linux-s390x.tar.gz}
 url=${GOURL:-$(urlFor $ver $file)}

 if test -e $build/bin && ! test -d $build/bin
```

* `/<source_root>/go-buildpack/manifest.yml`
```diff
@@ -2,7 +2,7 @@
 language: go
 default_versions:
 - name: go
-  version: 1.6.3
+  version: 1.6.1
 url_to_dependency_map:
 - match: go(\d+\.\d+(.*))
   name: go
@@ -15,45 +15,21 @@ url_to_dependency_map:
   version: v0.11.1
 dependencies:
 - name: go
-  version: 1.5.3
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/go/go1.5.3.linux-amd64.tar.gz
-  md5: 04b6198025b9a09889714d664bb94435
-  cf_stacks:
-  - cflinuxfs2
-- name: go
-  version: 1.5.4
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/go/go1.5.4.linux-amd64.tar.gz
-  md5: 27e19442233106d4deb329edc0980a46
-  cf_stacks:
-  - cflinuxfs2
-- name: go
-  version: 1.6.3
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/go/go1.6.3.linux-amd64.tar.gz
-  md5: 5f7bf9d61d2b0dd75c9e2cd7a87272cc
-  cf_stacks:
-  - cflinuxfs2
-- name: go
-  version: 1.6.2
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/go/go1.6.2.linux-amd64.tar.gz
-  md5: 6f1fd6eaf7d14b532e2a6c0e4840853b
-  cf_stacks:
-  - cflinuxfs2
-- name: go
-  version: 1.7
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/go/go1.7.linux-amd64.tar.gz
-  md5: 421bf4cc102f7ddee65c1ea377e3e47b
+  version: 1.6.1
+  uri: file:///<source_root>/go-buildpack/go1.6.linux-s390x.tar.gz
+  md5: d6e6ed2bea510e38c9926e4f994e854e
   cf_stacks:
   - cflinuxfs2
 - name: godep
   version: v74
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/godep/godep-v74-linux-x64.tgz
-  md5: 70220eee9f9e654e0b85887f696b6add
+  uri: file:///<source_root>/go-buildpack/godep-v74-linux-s390x.tgz
+  md5: bade891c3ffadc52cb591bedf3188163
   cf_stacks:
   - cflinuxfs2
 - name: glide
   version: v0.11.1
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/glide/glide-v0.11.1-linux-x64.tgz
-  md5: 122aec8824468d19ee468b433e8cdf8f
+  uri: file:///<source_root>/go-buildpack/glide-v0.11.1-linux-s390x.tgz
+  md5: e7b16132b9cf7a203130c554c690f31f
   cf_stacks:
   - cflinuxfs2
 exclude_files:
```
Here in `manifest.yml`, you need to carefully replace the dependencies with we built earlier before, and change the URI and MD5. The URI is the location of the dependencies and the MD5 value can be obtained by issuing an md5sum command as mentioned before. Some dependencies can have different versions, and you can simply remove their entries in the manifest if you don't need them in your buildpack. Or if you want them, you can build their binaries like mentioned before but with their corresponding versions. In our example, we only keep one version of each dependencies.

### Step 4 : Build Go Builpack
After all the changes, build the buildpack:
```
git submodule update --init
BUNDLE_GEMFILE=cf.Gemfile bundle
BUNDLE_GEMFILE=cf.Gemfile bundle exec buildpack-packager --cached
```
The dependency cached buildpack will be built as a zip file. To use it in your cloud foundry, after login to your cloud foundry instance, 
```
cf create-buildpack custom_go_buildpack go_buildpack-cached-custom.zip 1
cd /path_to_go_app
cf push go -b custom_go_buildpack
```
