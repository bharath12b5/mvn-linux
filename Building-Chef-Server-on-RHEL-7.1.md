# Building Chef-Server on RHEL 7.1

The Chef-Server code builds on RHEL 7.1 on IBM z System with this recipe. 

_**NOTE:** Use the root user throughout to build a RPM package for later installation._

### Before you begin:

1. You need a GitHub account (https://github.com/)

2. Register GitHub ssh-keys and account credentials on the current server.Please follow the instruction given in link below to generate ssh-keys

		https://help.github.com/articles/generating-ssh-keys/#platform-linux
	Run below command to configure GitHub on the current server

		yum install git.s390x
		git config --global user.email <GITHUB Email ID>
		git config --global user.name <GITHUB User Name>	

3. Install build-time dependencies:

	For Chef server:-

		yum install gcc.s390x readline-devel.s390 readline-devel.s390x  make.s390x unzip.s390x bzip2.s390x wget.s390x patch.s390x gcc-c++.s390x libtool.s390x rpm-build.s390x

	For Ruby :-

		yum install openssl.s390x openssl-devel.s390x zlib.s390x zlib-devel.s390x libffi.s390x libffi-devel.s390x

	_**NOTE:** Ruby installs without these dependencies, but does not function correctly._

4. Create a "Chef-Server-WorkSpace" directory for use as a working directory.

5. Install fakeroot.

	At the time of writing fakeroot is not available on RHEL 7.1 on IBM z System hence downloaded fakeroot & fakeroot-libs RPMs for fedora :

	Download and install RPMs from : http://www.rpmfind.net/linux/rpm2html/search.php?query=fakeroot

		wget ftp://195.220.108.108/linux/fedora-secondary/releases/20/Everything/s390x/os/Packages/f/fakeroot-libs-1.18.4-2.fc20.s390x.rpm
		wget ftp://195.220.108.108/linux/fedora-secondary/releases/20/Everything/s390x/os/Packages/f/fakeroot-1.18.4-2.fc20.s390x.rpm
		rpm -ivh fakeroot-libs-1.18.4-2.fc20.s390x.rpm
		rpm -ivh fakeroot-1.18.4-2.fc20.s390x.rpm	

6. Build & Install YAML

	Use wget to Download and install the YAML tar file from a URL:- 

		cd <Chef-Server-WorkSpace>
		wget http://pyyaml.org/download/libyaml/yaml-0.1.5.tar.gz
		tar -xvf yaml-0.1.5.tar.gz 
		cd yaml-0.1.5  
		./configure  
		make  
		make install 

7. Build & Install Ruby

	Ensure the Ruby Pre-Requisites mentioned in "Before you begin" section are installed **before** the Ruby build starts.
	Get Ruby (Ruby 2.1.4, built from source from https://www.ruby-lang.org/en/downloads

		cd <Chef-Server-WorkSpace> 
		wget http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.4.tar.gz
		tar -xvf  ruby
		./configure
		make  
		make install

	Verify Ruby is installed

		ruby -v

8. Build and install Ruby Gems

	Build and install Ruby Gems-2.2.2
	
		cd <Chef-Server-WorkSpace> 
		wget http://production.cf.rubygems.org/rubygems/rubygems-2.2.2.tgz
		tar -xvf  rubygems-2.2.2.tgz
		cd  rubygems-2.2.2    
		ruby setup.rb   

	Verify Ruby Gems installed

		gem env  
		gem install bundler 	

9. Install rake-compiler

		gem install rake-compiler  

  ### Building chef server:

  _**Note:** Ensure a GitHub Account, SSH Private key, and Account credentials are set as described in "Before you begin" section._

  Use Git Repository **opscode-omnibus** for the server, _(**Note:**-omnibus-chef is for the client only and that omnibus-software (subdirectory config/software) is the repository for ruby files to update as noted below)._

10. Create a base directory to download the Chef-Server source

		cd <Chef-Server-WorkSpace> 
		mkdir src
		cd src
		git clone https://github.com/chef/opscode-omnibus

	_**NOTE:** for a specific version, like 12.0.8 you can also download and unzip https://github.com/chef/opscode-omnibus/releases/tag/12.0.8)_

		git clone https://github.com/chef/omnibus-software       

	_**NOTE:** omnibus-software has no releases_

		cd opscode-omnibus  
		bundle install --binstubs  

11. You will need to modify some files before build

  1. First, copy below files from "omnibus-software" source tree  to the "opscode-omnibus" source tree leaving the original files in place.

		  cd \<Chef-Server-WorkSpace\>/src
		  cp -p  omnibus-software/config/software/ruby.rb       opscode-omnibus/config/software/ruby.rb
		  cp -p  omnibus-software/config/software/nodejs.rb     opscode-omnibus/config/software/nodejs.rb
		  cp -p  omnibus-software/config/software/sqitch.rb     opscode-omnibus/config/software/sqitch.rb
		  cp -p  omnibus-software/config/software/server-jre.rb opscode-omnibus/config/software/server-jre.rb
		  cp -p  omnibus-software/config/software/openresty.rb  opscode-omnibus/config/software/openresty.rb
		  cp -p  omnibus-software/config/software/cacerts.rb    opscode-omnibus/config/software/cacerts.rb

  2. Edit file "<Chef-Server-WorkSpace>/src/opscode-omnibus/files/private-chef-cookbooks/private-chef/recipes/opscode-solr4.rb" as given below.
	
	Replace 
	
		Xloggc   
	By
	
		Xverbosegclog   

	The changes can  be accomplished with the following 'sed' script.
	
		sed -i 's/Xloggc/Xverbosegclog/g'  files/private-chef-cookbooks/private-chef/recipes/opscode-solr4.rb  
		
	(Reason:- Allows IBM Java to understand Xloggc and start.)


  3. Edit file "<Chef-Server-WorkSpace>/src/opscode-omnibus/omnibus.rb" as given below.

	  Replace
	
		  #use_s3_caching true  
		  #s3_access_key  ENV['AWS_ACCESS_KEY_ID']  
		  #s3_secret_key  ENV['AWS_SECRET_ACCESS_KEY']  
		  #s3_bucket      'opscode-omnibus-cache'  
	  By
	
		  use_s3_caching false   
		
	(Reason:- turn off S3 caching to enable a full true build)


  4. Edit File <Chef-Server-WorkSpace>/src/opscode-omnibus/config/software/openresty-lpeg.rb

	Replace  _**Note:** luajit-2.0 might be seen as luajit-2.1, etc._ 
	
		make "LUADIR=#{install_dir}/embedded/luajit/include/luajit-2.0", env: env		
	By
	
		make "LUADIR=#{install_dir}/embedded/lua/include/", env: env
		
	(Reason:-  change LUADIR location because jit is not installed)

  5. Edit File <Chef-Server-WorkSpace>/src/opscode-omnibus/config/software/opscode-solr4.rb

	Replace lucene/solr url.
	
		source url: "http://www.dsgnwrld.com/am/lucene/solr/4.5.1/solr-#{version}.tgz",
	By
	
		source url: "https://archive.apache.org/dist/lucene/solr/4.5.1/solr-#{version}.tgz",   
		
	(Reason:- The previous url is no longer reachable) 


  6. Edit File <Chef-Server-WorkSpace>/src/opscode-omnibus/config/software/libossp-uuid.rb

	Replace  _**Note:** The md5 sum does not apper to have changed in this instance_
	
		source url: "ftp://ftp.ossp.org/pkg/lib/uuid/uuid-#{version}.tar.gz"
		md5: "5db0d43a9022a6ebbbc25337ae28942f"		
	By
	
		source url: "https://gnome-build-stage-1.googlecode.com/files/uuid-1.6.2.tar.gz",   
		md5: "5db0d43a9022a6ebbbc25337ae28942f"   
		
	(Reason:- ftp server no longer available)


  7. Edit File <Chef-Server-WorkSpace>/src/opscode-omnibus/config/software/ruby.rb

	Replace 
	
		"--disable-dtrace"]   		
	By
	
		"--disable-dtrace",   
		"--with-setjmp-type=_setjmp"]   
	
	(Reason :- Server will crash without "--with-setjmp-type=_setjmp" )

  8. Edit File <Chef-Server-WorkSpace>/src/opscode-omnibus/config/software/nodejs.rb

	Replace the entire body of nodejs.rb with  the following :- 
	
		name "nodejs"   
		default_version "0.10.36"  
		dependency "python"   
		version "0.10.36" do   
		source md5: "02de00cb56c976f71a5a9eb693511fe7"   
		end  
		source url: "https://github.com/andrewlow/node/archive/V8-3.14.5.9-Node.js-#{version}-201501281023.tar.gz"   
		relative_path "node-8-3.14.5.9-Node.js-#{version}-201501281023"   
		build do   
		env = with_standard_compiler_flags(with_embedded_path)   
		env["LINK"]="g++"   
		command "#{install_dir}/embedded/bin/python ./configure" \   
		" --dest-cpu=s390x --prefix=#{install_dir}/embedded", env: env   
		make "-j #{workers}", env: env   
		make "install", env: env   
		end

	(Reason:-  use opensource node.js modified for Z ) 
	
	_**NOTE:** in the following, the line_   
	
		env["LINK"]="g++"   
		
	_is optional -- it's only needed if you are building into an nfs mounted filesystem
	(without it in that circumstance, flock throws an error)_

  9. Edit File <Chef-Server-WorkSpace>/src/opscode-omnibus/config/software/sqitch.rb

	Replace 
	
		default_version "0.973"
	By
	
		default_version "0.999"

	...and Replace 
	
		md5: "0994e9f906a7a4a2e97049c8dbaef584"
	
	By
	
		md5: "b3a9cac1254e0e90e4cc09fc84a66c93"
	
	(Reason :- The old version is no longer available)

  10. Edit File <Chef-Server-WorkSpace>/src/opscode-omnibus/config/software/server-jre.rb
	
	Replace the entire body of server-jre.rb with  the following :- 
	
		name "server-jre"
		dependency "rsync"
		jre_dir = "#{install_dir}/embedded/jre"
		build do
		command "wget http://tlbgsa.ibm.com/<YourID>/public/ibm-java-jre-7.0-7.1-s390x.tgz"
		command "mkdir -p #{jre_dir}"
		command "tar xzf ibm-java-jre-7.0-7.1-s390x.tgz"
		command "#{install_dir}/embedded/bin/rsync -a ./jre #{install_dir}/embedded"
		command "chmod +x #{install_dir}/embedded/jre/bin/*"
		end
		
	(Reason:-  There is no Z "server-jre" available on the IBM site, so simply use  the IBM jre. The server-jre.rb copies the jre into the right directories).

  11. Edit File <Chef-Server-WorkSpace>/src/opscode-omnibus/config/software/openresty.rb

	in the sequence of build parameters, take out all the jits and asms (not present on Z)
	Replace  
	
		'--with-luajit',
	By
	
		'--with-lua51',
	
	Remove the following lines completely :- 
	
		'--with-md5-asm',
		'--with-sha1-asm',
		'--with-pcre-jit',


  12. Edit File <Chef-Server-WorkSpace>/src/opscode-omnibus/config/software/cacerts.rb

	Replace

		version "2014.08.20" do
		source md5: "c9f4f7f4d6a5ef6633e893577a09865e"
		end
	By
	
		version "2014.08.20" do
		source md5: "380df856e8f789c1af97d0da9a243769"
		end

  13. Two files in package-scripts/chef-server needs to be changed: post installation**

	We need to modify couple of files in the ohai package (cpu.rb and platform.rb). The purpose of these modifications is to enable ohai to properly sense and report on the Z/390 platform. Just add the following lines just before the block of "next-step/welcome" messages in the postinst file

		patch -p0 << END_OF_FILE
		--- /opt/opscode/embedded/service/gem/ruby/2.1.0/gems/ohai-6.22.0/lib/ohai/plugins/linux/cpu.rb 2015-03-18 16:03:46.857938737 +0100
		+++ /opt/opscode/embedded/service/gem/ruby/2.1.0/gems/ohai-6.22.0/lib/ohai/plugins/linux/cpu.rb 2015-03-18 16:09:58.097942687 +0100
		@@ -30,7 +30,12 @@
		current_cpu = \$1
		cpu_number += 1
		when /vendor_id\s+:\s(.+)/
		-    cpuinfo[current_cpu]["vendor_id"] = \$1
		+    vendor_id = \$1
		+    if vendor_id =˜ (/IBM\/S390/)
		+      cpuinfo["vendor_id"] = vendor_id
		+    else
		+      cpuinfo[current_cpu]["vendor_id"] = vendor_id
		+    end
		when /cpu family\s+:\s(.+)/
		cpuinfo[current_cpu]["family"] = \$1
		when /model\s+:\s(.+)/
		@@ -52,6 +57,21 @@
		cpuinfo[current_cpu]["cache_size"] = \$1
		when /flags\s+:\s(.+)/
		cpuinfo[current_cpu]["flags"] = \$1.split(' ')
		+  when /bogomips per cpu:\s(.+)/
		+    cpuinfo["bogomips per cpu"] = \$1
		+  when /features\s+:\s(.+)/
		+    cpuinfo["features"] = \$1.split(' ')
		+  when /processor\s(\d):\s(.+)/
		+    current_cpu = \$1
		+    cpu_number += 1
		+    cpuinfo[current_cpu] = Mash.new
		+    current_cpu_info = \$2.split(',')
		+    for i in current_cpu_info
		+      name_value = i.split('=')
		+      name = name_value[0].strip
		+      value = name_value[1].strip
		+      cpuinfo[current_cpu][name] = value
		+    end
		end
		end
		END_OF_FILE

		patch -p0 << END_OF_FILE
		--- /opt/opscode/embedded/service/gem/ruby/2.1.0/gems/ohai-6.22.0/lib/ohai/plugins/linux/platform.rb    2015-03-18 11:04:30.217941332 +0100
		+++ /opt/opscode/embedded/service/gem/ruby/2.1.0/gems/ohai-6.22.0/lib/ohai/plugins/linux/platform.rb    2015-03-18 11:05:31.607942099 +0100
		@@ -100,7 +100,7 @@
		platform_family "debian"
		when /fedora/
		platform_family "fedora"
		-  when /oracle/, /centos/, /redhat/, /scientific/, /enterpriseenterprise/, /amazon/, /xenserver/ # Note that 'enterpriseenterprise' is oracle's LSB "distributor ID"
		+  when /oracle/, /centos/, /redhat/, /scientific/, /enterpriseenterprise/, /amazon/, /xenserver/, /base/ # Note that 'enterpriseenterprise' is oracle's LSB "distributor ID"
		platform_family "rhel"
		when /suse/
		platform_family "suse"
		END_OF_FILE

### Kick off the build
	
		bin/omnibus build chef-server --log-level=debug --override append_timestamp:false    

This gives copious debugging information plus produces an rpm that doesn't have a timestamp

_**Note:** At the time of writing this recipe , the cacert.pem file used by opscode-omnibus build is out of date and the checksum is not matching with the new certificate available at(http://curl.haxx.se/docs/caextract.html) The workaround this problem is to use the old cacert.pem file ( please find at <>) and follow the steps below:_

		export SSL_CERT_FILE= <path to downloaded cacert.pem
		cd  Chef-Server-WorkSpace>/src/opscode-omnibus
		Remove the Gem.lock file from opscode directory
		bundle install --binstubs 

_This workaround will not be needed once the opscode-omnibus code build is fixed to use the latest certificate ._

### The Whitelist

After the build finishes building its components, it does a health check. It may fail the health check because of missing items on the "whitelist"
You will now find a directory named something like....
/usr/local/lib/ruby/gems/2.1.0/bundler/gems/omnibus-51ca37c7b299/lib/omnibus/

	containing a file health_check.rb
	_**NOTE:** the directory may have different alphanumerics in its name than 51ca37c7b299_

	In health_check.rb you need to add the following list of files to the whitelist (after the items already there)

		WHITELIST_LIBS = [
		. . .
		/libmawt\.so/,
		/libjvm\.so/,
		/libj9thr26\.so/,
		/libj9hookable26\.so/,
		/libXext\.so/,
		/libX11\.so/,
		/libXrender\.so/,
		/libXft\.so/,
		/libXtst\.so/,
		/libXi\.so/,
		/libfreetype\.so/,
		/libXau\.so/,
		/libexpat\.so/,
		/libfontconfig\.so/,
		/libxcb\.so/,
		/libreadline\.so/,
		/libhistory\.so/
		].freeze

### After building, install the rpm to test it.

The server is controlled with the command chef-server-ctl

	To start:

		chef-server-ctl start    

	To test:

		chef-server-ctl test
	