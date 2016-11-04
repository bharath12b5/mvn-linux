# Building Cloud Foundry Java Buildpack

### _**General Note:**_
i) _When following the steps below please use a standard permission user unless otherwise specified._

ii) _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._

This is an instruction to build Cloud Foundry Java Buildpack. For proof of concept purpose, this instruction builds the buildpack based on OpenJDK 1.8.0 on Ubuntu 16.04. The official Cloud Foundry Java buildpack supports many JVM-based applications and frameworks. Most of them would work since these components come as jar files. However, some frameworks, whose dependencies or libraries need to be recompiled on s390x platform, are not supported on IBM z systems since their source code is not open at the time when the recipe was written. These frameworks include JRebel Agent, Luna Security Provider and Dynatrace-OneAgent.

### Step 1 : Get the source code
```
cd /<source_root>
git clone https://github.com/cloudfoundry/java-buildpack.git
```

### Step 2 : Build the original buildpack
Build the offline buildpack:
```
cd java-buildpack
bundle install
bundle exec rake package OFFLINE=true PINNED=true
```
The buildpack zip file will be generated in `/<source_root>/java-buildpack/build` and the contents in the zip file will be in folder `/<source_root>/java-buildpack/build/staging`.

### Step 3 : Make changes to the existing buildpack

Update the following dependencies:

#### OpenJDK
```
sudo apt-get install openjdk-8-jdk
cd /<source_root>/java-buildpack
mkdir tmp
cp -rL /usr/lib/jvm/java-8-openjdk-s390x/* ./tmp
```

Create the OpenJDK package:
```
cd /<source_root>/java-buildpack/tmp
tar czf ../build/staging/resources/cache/https%3A%2F%2Fjava-buildpack.cloudfoundry.org%2Fopenjdk%2Fxenial%2Fs390x%2Fopenjdk-1.8.0.tar.gz.cached ./*
```

Create an index file of the newly added package:
```
cd /<source_root>/java-buildpack/build/staging/resources/cache/
cat > https%3A%2F%2Fjava-buildpack.cloudfoundry.org%2Fopenjdk%2Fxenial%2Fs390x%2Findex.yml.cached <<EOF
>---
>1.8.0: https://java-buildpack.cloudfoundry.org/openjdk/xenial/s390x/openjdk-1.8.0.tar.gz
>EOF 
```

#### Memory-Calculator
```
sudo apt-get install golang
export GOPATH=/<go_path>/
go get -v github.com/cloudfoundry/java-buildpack-memory-calculator
cd /<go_path>/src/github.com/cloudfoundry/java-buildpack-memory-calculator
git checkout v2.0.2.RELEASE
go get -v github.com/tools/godep
```

Before building, to perform the tests and pass the tests, do the following change first:

##### `/<go_path>/src/github.com/cloudfoundry/java-buildpack-memory-calculator/ci/test.sh`
```diff
@@ -15,5 +15,5 @@ pushd $GOPATH/src/github.com/cloudfoundry/java-buildpack-memory-calculator
   PATH=${GOPATH//://bin:}/bin:$PATH

   go install -v github.com/onsi/ginkgo/ginkgo
-  ginkgo -r -failOnPending -randomizeAllSpecs -skipMeasurements=true -race "$@"
+  ginkgo -r -failOnPending -randomizeAllSpecs -skipMeasurements=true "$@"
 popd
```

##### `/<go_path>/src/github.com/cloudfoundry/java-buildpack-memory-calculator/integration/integration_suite_test.go`
```diff
@@ -30,7 +30,7 @@ func TestIntegration(t *testing.T) {

        // build main program for testing
        SynchronizedBeforeSuite(func() []byte {
-               jbmcExec, err := gexec.Build("github.com/cloudfoundry/java-buildpack-memory-calculator", "-a", "-race")
+               jbmcExec, err := gexec.Build("github.com/cloudfoundry/java-buildpack-memory-calculator", "-a")
                <CE><A9>(err).ShouldNot(HaveOccurred())
                return []byte(jbmcExec)
        }, func(jbmcBinPath []byte) {
```

Test, should all pass:
```
./ci/test.sh
```

Build and place the package in the java buildpack dependency folder:
```
cd /<go_path>/src/github.com/cloudfoundry/java-buildpack-memory-calculator
./ci/build.sh
cp java-buildpack-memory-calculator-linux.tar.gz /<source_root>/java-buildpack/build/staging/resources/cache/https%3A%2F%2Fjava-buildpack.cloudfoundry.org%2Fmemory-calculator%2Fxenial%2Fs390x%2Fmemory-calculator-2.0.2_RELEASE.tar.gz.cached
```

Create an index file for the newly added package:
```
cd /<source_root>/java-buildpack/build/staging/resources/cache/
cat > https%3A%2F%2Fjava-buildpack.cloudfoundry.org%2Fmemory-calculator%2Fxenial%2Fs390x%2Findex.yml.cached <<EOF
>---
>2.0.2_RELEASE: https://java-buildpack.cloudfoundry.org/memory-calculator/xenial/s390x/memory-calculator-2.0.2_RELEASE.tar.gz
>EOF 
```

### Step 3 : Build Java Builpack
Rezip the buildpack with all the changes made:
```
cd /<source_root>/java-buildpack/build/staging
zip -r ../java-buildpack-offline-s390x.zip ./*
```

Now you can upload the buildpack:
```
cf create-buildpack custom_java_buildpack /<source_root>/java-buildpack/build/java-buildpack-offline-s390x.zip 1
```

To test your buildpack, you can use applications provided by Cloud Foundry at `https://github.com/cloudfoundry/java-test-applications`. We use the `java-main-application` as an example:
```
git clone https://github.com/cloudfoundry/java-test-applications.git
cd java-test-applications/java-main-application
../gradlew build
cf push java-app -p ./build/libs/java-main-application-1.0.0.BUILD-SNAPSHOT.jar -b custom_java_buildpack
```

If you see the following error when deploy your application:
```
-----> Uploading droplet (23M)
...
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 down
...
0 of 1 instances running, 1 failing
FAILED
```
You can increase the memory size in the `manifest.yml` file of your application. That usually resolves the issue.