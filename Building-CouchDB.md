The [CouchDB 2.0 Developer Preview](https://couchdb.apache.org/developer-preview/2.0/) can be built on RHEL 7 on IBM z System by following these instructions.

1. First, install Erlang according to the instructions in [Building Erlang](../Building-Erlang).

2. As root, add the RHEL 7 Server Optional repository to the yum repository list. This is required to get the js-devel package. See the instructions in [Adding RHEL Optional and Supplementary Repositories](../Adding-RHEL-Optional-and-Supplementary-Repositories).

3. As root, install all the build-time dependencies:

		yum install nspr-devel icu-devel libcurl-devel js js-devel

4. Download IBM SDK for Node.js from [developerWorks](http://www.ibm.com/developerworks/web/nodesdk/). Be sure to select the binary for "Linux on System z". Run the downloaded interactive installer as root, and specify /opt/ibm/node/ as the installation directory.

		./node-v0.10.35-linux-s390x.bin

5. As a regular user, set the PATH environment variable to include all required directories:

		export PATH=/opt/ibm/node/bin:/opt/erlang/bin:/usr/local/bin:$PATH

6. Install Grunt using NPM:

		npm install -g grunt-cli

7. Building CouchDB requires [rebar](https://github.com/rebar). Build it from source:

		git clone git://github.com/rebar/rebar.git
		cd rebar
		./bootstrap

8. The `bootstrap` command produces an Erlang escript named rebar. Copy it into a directory in your $PATH:

		cp rebar /usr/local/bin/

9. Now set up the CouchDB source tree:

		git clone https://git-wip-us.apache.org/repos/asf/couchdb.git
		cd couchdb
		git checkout developer-preview-2.0

   Replace the branch name "developer-preview-2.0" with "master" if you wish to build the latest version of CouchDB.
  
10. Configure the source tree (which will fetch more source code from version control), then run `make` to build CouchDB:

		./configure -p /opt/couchdb -c
		make

   Alternatively, run `make check` to build CouchDB and run also some sanity tests.

11. Verify that the build works by starting the test server (you can kill it with Ctrl-C):

		dev/run

12. Generate an installable release with an embedded Erlang run-time system:

		make dist

   The rel/ sub-directory should now contain a complete CouchDB installation. It can be installed simply by moving (as root):

		cp -R rel/couchdb /opt/couchdb
