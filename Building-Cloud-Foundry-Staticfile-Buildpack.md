# Building Cloud Foundry Staticfile Buildpack

### _**General Note:**_
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

This is an instruction to build Cloud Foundry Staticfile Buildpack.

### Step 1 : Get the source code
```
cd /<source_root>
git clone https://github.com/cloudfoundry/staticfile-buildpack.git
```

### Step 2 : Build necessary dependencies

#### 1. [Build warden root filesystems](https://github.com/linux-on-ibm-z/docs/wiki/Building-Cloud-Foundry-Ruby-Buildpack#1-build-warden-root-filesystems)

#### 2. [Prepare Cloud Foundry Binary-Builder](https://github.com/linux-on-ibm-z/docs/wiki/Building-Cloud-Foundry-Ruby-Buildpack#2-prepare-cloud-foundry-binary-builder)

#### 3. NGINX

We use Cloud Foundry binary-builder to build NGINX.
```
cd /<source_root>/binary-builder
```

Run the binary-builder to build the binary:
```
cd /<source_root>/binary-builder
docker run -w /binary-builder -v `pwd`:/binary-builder -it cloudfoundry/cflinuxfs2 ./bin/binary-builder --name=nginx --version=1.11.4 --md5=92666d60410f429ddbdb10060369a480
cp nginx-1.11.4-linux-s390x.tgz /<source_root>/staticfile-buildpack
```
    
Note the binary-builder will download the source code from `http://nginx.org/download/nginx-1.11.4.tar.gz`, so the MD5 hash code is provided in the command to provide authenticity and integrity of the downloaded package.

### Step 3 : Change the following files:

* `/<source_root>/staticfile-buildpack/manifest.yml`
```diff
@@ -17,7 +17,7 @@ url_to_dependency_map:
 dependencies:
 - name: nginx
   version: 1.11.4
-  uri: https://buildpacks.cloudfoundry.org/concourse-binaries/nginx/nginx-1.11.4-linux-x64.tgz
-  md5: 5a62aad3f1d7baf8e47c8a61d710848e
+  uri: file:///<source_root>/staticfile-buildpack/nginx-1.11.4-linux-s390x.tgz
+  md5: fc9b3db31ba00f1a3dfe94f6a3290a85
   cf_stacks:
   - cflinuxfs2
```
Here in `manifest.yml`, you need to carefully replace the dependencies which we built earlier before, and change the URI and MD5. The URI is the location of the dependencies and the MD5 value can be obtained by issuing an md5sum command as mentioned before.

### Step 4 : Build Staticfile Builpack
After all the changes, build the buildpack:
```
git submodule update --init
BUNDLE_GEMFILE=cf.Gemfile bundle
BUNDLE_GEMFILE=cf.Gemfile bundle exec buildpack-packager --cached
```
The dependency cached buildpack will be built as a zip file. To use it in your cloud foundry, after login to your cloud foundry instance, 
```
cf create-buildpack custom_staticfile_buildpack staticfile_buildpack-cached-custom.zip 1
cd /path_to_staticfile_app
cf push staticfile_app -b custom_staticfile_buildpack
```
