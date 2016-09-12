<!---PACKAGE:Chef Server--->
<!---DISTRO:RHEL 6.6:12.7--->
<!---DISTRO:RHEL 7.1:12.7--->
<!---DISTRO:SLES 11:12.7--->
<!---DISTRO:SLES 12:12.7--->
<!---DISTRO:Ubuntu 16.x:12.7--->

The instructions provided below specify the steps to build Chef Server Omnibus 12.7.0 on Linux on the IBM z Systems for RHEL 6/7, SLES 11/12 and Ubuntu 16.04.

_**General Notes:**_ 	
_i) When following the steps below please use a super user / root. This isn't best practice (but the build process is much more stable as a super user) however this RPM could be built on a VM / Container so that root isn't exposed._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it. During the build the environment variable `$SRCRT` will be set to hold this and make the recipe easier to follow_  
_iii) For convenience `vi` has been used in the instructions below when editing files, replace with your desired editing program if required_

## Building Chef Server Omnibus

1. Install the build dependencies

    On RHEL 7.1 systems
    ```shell
    yum install git ruby-devel rubygems make gcc gcc-c++ patch readline-devel bzip2 tar java-1.7.1-ibm-devel libxml2-devel which wget curl rpm-build
    ```
    On RHEL 6.6 systems:
    ```shell
    yum install git gcc gcc-c++ bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel ncurses-devel make wget tar patch bzip2 java-1.7.1-ibm-devel which libxml2-devel curl rpm-build
    ```
    On SLES 12 systems
    ```shell
    zypper install git make gcc tar gcc-c++ patch readline-devel bzip2 java-1_7_1-ibm-devel which wget libxml2-devel curl timezone rpm-build libopenssl-devel bison flex libyaml-devel libffi48-devel zlib-devel ncurses-devel
    ```
    On SLES 11 systems
    ```shell
    zypper install git make gcc47 tar gcc47-c++ bison flex libopenssl-devel libyaml-devel libffi-devel readline-devel zlib-devel ncurses-devel wget autoconf patch bzip2 java-1_7_0-ibm libxml2-devel curl timezone
    ```
   On Ubuntu 16.04
    ```shell
	apt-get update
    apt-get install git make gcc tar g++ patch libreadline6 libreadline6-dev bzip2 openjdk-8-jdk wget libxml2-dev curl libssl-dev bison flex ruby libyaml-dev libffi-dev zlib1g-dev libncurses5-dev ruby-dev 
    ```
    You may already have some of these packages installed - install any that are missing if needed.  
    _**Note:** The `tcl-devel`, `tk-devel` and `gdbm-devel` packages should NOT be installed (otherwise the Ruby build process will incorporate them and Chef Server Omnibus may fail with a healthcheck dependency error later)_

2. Install other build dependency 
  
  a. Only for **RHEL6** : **git >= 1.8.5**  
      _Git version 1.8.5 or later is required to build this version of Chef-Server. Update git version >= 1.8.5 by  building it from source._

   1. _[Optional]_ Check the version of any existing git executable.
        ```
           which git
           $(which git) --version
        ```   
        _**Note:** If above command gives git version lesser than 1.8.5, then follow below steps_  

   2. Install build dependencies for git
        ```
           sudo yum install -y asciidoc-8.6.8-5.el7.noarch openssl-devel curl-devel expat-devel perl-ExtUtils-MakeMaker gettext-devel tar
        ```  

   3. Download the git source code
        ```
           cd /<source_root>/
           git clone https://github.com/git/git.git
           cd /<source_root>/git/
           git checkout v1.8.5
        ```  

   4. Build and Install git
        ```
           make prefix=<build-location> && sudo make prefix=<build-location> install 
        ```  

   5. Export git binary path to PATH environment variable
        ```
	   export PATH=<build-location>/bin:$PATH  
        ```
	
  b. Only for **SLES11** : **openSSL >= 1.0.1** 
        
   ```
      cd /<source_root>/  
      git clone https://github.com/openssl/openssl.git  
      cd openssl  
      git checkout OpenSSL_1_0_1t  
      ./config --prefix=/usr --openssldir=/usr/local/openssl shared  
      make  
      sudo make install
   ```  

3. Correct the gcc linking for SLES 11 with gcc 4.3 **only**

      ```shell
    ln -sf /usr/bin/cpp-4.7 /usr/bin/cpp
    ln -sf /usr/bin/g++-4.7 /usr/bin/g++
    ln -sf /usr/bin/gcc-4.7 /usr/bin/gcc
    ln -sf /usr/bin/gcc-ar-4.7 /usr/bin/gcc-ar
    ln -sf /usr/bin/gcc-nm-4.7 /usr/bin/gcc-nm
    ln -sf /usr/bin/gcc-ranlib-4.7 /usr/bin/gcc-ranlib
    ln -sf /usr/bin/gcc /usr/bin/cc
    ln -sf /usr/bin/g++ /usr/bin/c++
    ```
    _**Note:** This is necessary because the standard gcc version available on SLES 11 is 4.3, and chef requires 4.4+ to build, but the 4.7 gcc package available on SLES 11 suffixes all the executables._

4. Build Ruby from source (**only for RHEL 7.1 / RHEL 6.6 / SLES 11 / SLES 12**)

    **WARNING:** Do not perform the Ruby install dependencies step as that specifies the tcl / tk / gdbm libraries. The dependencies above cover the Ruby build process. If you already have tcl-devel, tk-devel or gdbm-devel installed you'll need to remove them for the duration of this build.
    
    *For RHEL7.1 and SLES12:*  
    The Ruby version available on SLES 12 has some configuration issues so build and install Ruby yourself following the instructions [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby).

    *For RHEL6.6 and SLES11:*  
    The Ruby version available on the RHEL 6.6 and SLES11 package repositories is too low level and the ruby version from above link gives some build issues for RHEL 6.6 and SLES 11. Hence build Ruby version 2.2.5 by using the above [link](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby) as reference.

    _**Note:** While installing Ruby on **RHEL6** and **SLES11** if **GCC 6.0.0**  was installed, make sure while performing the below steps **GCC 6.0.0** version **should not** be used. It causes some compilation error for ncurses._
    
    Once you have completed the Ruby installation continue to follow the process below
5. Correct the gem environment and install bundler

	On RHEL/SLES:
	
    ```shell
    export GEM_HOME=<USER_HOME_DIR>/.gem/ruby
    export PATH=$GEM_HOME/bin:$PATH
    gem install bundler
    ```
    _Where `<USER_HOME_DIR>` is the home directory of the user you are installing under._  

    On Ubuntu 16.04:

    ```shell
    gem install bundler
    ```

    _**Note:** Run `gem env` to verify the state of the environment, if later on you have issues installing / running ruby gems please ensure the environment is set correctly._
6. Setup `<source_root>` and `$SRCRT`

    ```shell
    mkdir /<source_root>/
    cd /<source_root>
    export SRCRT=`pwd`
    ```
    _**Note:** You can use an existing directory for `<source_root>` but this might make clean-up a little more awkward afterwards_
7. Download the build process code

    ```shell
    cd $SRCRT
    git clone --depth 100 https://github.com/chef/omnibus-software
    git clone https://github.com/chef/chef-server
    git clone https://github.com/chef/omnibus
    cd omnibus
    git checkout v5.5.0
    cd ..
    ```
    _**Note:** The `--depth 100` flag is used to reduce the amount downloaded, in the future this might mean that specific checkouts won't work (as the total number of commits has moved on), in that case either increase the value or omit the tag_  

    Only the **Ubuntu** platforms require modification.
    ```shell
    vi omnibus/lib/omnibus/packager.rb
    ```
    Update the packaging list to understand Ubuntu platforms as follows:
	
	From
	```ruby
	 PLATFORM_PACKAGER_MAP = {
      "debian"   => DEB,
      "fedora"   => RPM,
      "suse"     => RPM,
      "rhel"     => RPM,
      "wrlinux"  => RPM,
      "aix"      => BFF,
      "solaris"  => Solaris,
      "ips"      => IPS,
      "windows"  => [MSI, APPX],
      "mac_os_x" => PKG,
    }.freeze
	```
	to
    ```ruby
    PLATFORM_PACKAGER_MAP = {
      'debian'   => DEB,
      'fedora'   => RPM,
      'rhel'     => RPM,
      'suse'     => RPM,
      'aix'      => BFF,
      'solaris2' => Solaris,
      'windows'  => MSI,
      'mac_os_x' => PKG,
      'ubuntu'   => DEB,
    }.freeze
    ```
    _**Note:** `'ubuntu'` was added and pointing to DEB_  
    This is because the omnibus gem doesn't support / recognize `'ubuntu'` as a platform and so drops into a `makeself` package.
8. Checkout the required version of Chef Server and download the Ruby Gem requirements

    ```shell
    cd $SRCRT/chef-server/
    git checkout 12.7.0
    cd omnibus
    vi Gemfile
    ```
    a. Update the Gemfile to add a couple of missing Gems and point omnibus to the git repo downloaded earlier. 
    
	From 
	```ruby
	gem 'rake'
	gem 'chefspec'
	gem 'berkshelf'

	# Install omnibus software
	group :omnibus do
		gem 'omnibus', github: 'chef/omnibus'
		gem 'omnibus-software', github: 'chef/omnibus-software'
	end

	group :test do
		gem 'rspec'
	end
	```
	to
	```ruby
	gem 'rake'
	gem 'chefspec'
	gem 'berkshelf'
	gem 'json'
    gem 'json_pure'
    
	# Install omnibus software
	group :omnibus do
		gem 'omnibus', path: '/<source_root>/omnibus'
		gem 'omnibus-software', github: 'opscode/omnibus-software'
	end

	group :test do
		gem 'rspec'
	end
	```
    _**Note:** Add the `json` and `json_pure` gems on all platforms, change the `gem 'omnibus'` line. Also point the omnibus-software to the 'opscode/omnibus-software'._  
    
    b. Install following missing Gems only for **RHEL** 
    ```shell
    gem install mixlib-shellout
    gem install chef-sugar
    gem install ohai
    gem install aws-sdk
    ```
9. Finally install the build time Gems and configure git with an email.
    ```shell
    bundle install --binstubs
    git config --global user.email "you@example.com"
    ```
    _**Note:**_
	
    _i) Git needs to be configured with at least a user.email (if this is already set then there is no need to update it) in order for the git caching to work - the caching is useful to recover from missing any steps without having to do a complete rebuild. If you'd prefer not to have to configure git you can disable git caching in **10.i**._

    _ii) If you get the following error "An error occurred while installing dep-selector-libgecode (1.0.2), and Bundler cannot continue.
    Make sure that `gem install dep-selector-libgecode -v '1.0.2'` succeeds before bundling."
    Follow the steps mentioned below:_
    
    _Manually install gecode (for RHEL7/Ubuntu 16.04 only):_
    ```shell
    apt-get install libqtcore4  libqt4-dev qt4-qmake
    wget http://www.gecode.org/download/gecode-3.7.3.tar.gz /
    tar -xvf gecode-3.7.3.tar.gz
    cd gecode-3.7.3
    ./configure && make && make install
    USE_SYSTEM_GECODE=1 gem install dep-selector-libgecode
    export USE_SYSTEM_GECODE=1
    ```

    _iii) If you get the following error "An error occurred while installing rack (1.6.4), and Bundler cannot continue.
    Make sure that `gem install rack -v '1.6.4'` succeeds before bundling."._

    _Install the gem manually as mentioned below (for SLES11 only):_
     ```shell
     gem install rack
     ```
10. Update a number of different files

    1. Update omnibus.rb
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        vi omnibus.rb
        ```
        Turn off s3 caching (we need to build a number of items differently from the s3 cached version), and optionally turn off git caching (it is recommended to keep git caching on).
        
		From
		```ruby
        use_s3_caching true
		```
		to
		```ruby
        use_s3_caching false
		```
        _**Note:** set `false` for `use_s3_caching`._
    2. Update chef-server.rb
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        vi config/projects/chef-server.rb
        ```
        Here we need to update the version of Ruby bundled - the `2.1.4` version of Ruby has some SSL issues:
        
		From
		```ruby
		override :ruby, version: "2.1.4"
		```
		to
		```ruby
		override :ruby, version: "2.2.2"
		```
        _**Note:**  change the `:ruby` version to be `2.2.2`_
    3. Update openresty.rb
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/openresty.rb config/software/.
        vi config/software/openresty.rb
        ```
        Copy the default `openresty.rb` file in order to be able to make the changes otherwise the build process will download a fresh copy. Two changes are required to be done in the file as mentioned below.
        
		From
		```ruby
		# Options inspired by the OpenResty Cookbook
		"--with-md5-asm",
		"--with-sha1-asm",
		"--with-pcre-jit",
		"--without-http_ssi_module",
		```
		to
		```ruby
        # Options inspired by the OpenResty Cookbook
        #'--with-md5-asm',
        #'--with-sha1-asm',
        #'--with-pcre-jit'
        '--without-http_ssi_module'
        ```
		and from
		```ruby
		# Currently LuaJIT doesn't support POWER correctly so use Lua51 there instead
		if ppc64? || ppc64le? || s390x?
			configure << "--with-lua51=#{install_dir}/embedded/lib"
		else
			configure << "--with-luajit"
		end
		```
		to
        ```ruby
		# Currently LuaJIT doesn't support POWER correctly so use Lua51 there instead
		if ppc64? || ppc64le? || s390x?
			configure << "--with-lua51=#{install_dir}/embedded/lib"
		else
			configure << "--with-lua51"
		end
		```
		
    4. Update opscode-solr4.rb

		```shell
		cd $SRCRT/chef-server/omnibus/
		vi config/software/opscode-solr4.rb
		```
		changes are required to be done in the file as mentioned below.
		
		From
		```ruby
		if ppc64? || ppc64le?
			dependency "ibm-jre"
		elsif intel? && _64_bit?
			dependency "server-jre"
		else
			raise "A JRE is required by opscode-solr4, but none are known for this platform"
		end
		```
		to
		```ruby
		    dependency "server-jre"
		```

    5. Update server-jre.rb
   
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/server-jre.rb config/software/.
        ```
      On RHEL/SLES With IBM JDK :
       ```shell
        rpm -qa | grep java | xargs rpm -ql | grep "\/java$" | grep jre | sed "s#jre/bin/java##"
        vi config/software/server-jre.rb
        ```
        The `rpm -qa...` command will produce a path that points to the base of the installed IBM JDK, this path will be needed in the editing of the `server-jre.rb` file
        ```ruby
        name "server-jre"
        default_version "1.7.<x>"
        raise "Server-jre can only be installed on x86_64 systems." unless _64_bit?
        
        whitelist_file "jre/bin/javaws"
        whitelist_file "jre/bin/policytool"
        whitelist_file "jre/lib"
        whitelist_file "jre/plugin"
        whitelist_file "jre/bin/appletviewer"
        
        version "1.7.<x>" do
            source path:  "<path_above>"
        end
        
        build do
            mkdir "#{install_dir}/embedded/jre"
            sync  "#{project_dir}/", "#{install_dir}/embedded/jre"
        end
        ```
_**Note:** Where `<x>` is the version of java (1.7.1 or 1.7.0 depending on platform) and `<path_above>` is the path returned by the `rpm -qa...` command run earlier._

     On Ubuntu 16.04:
        ```shell
         vi config/software/server-jre.rb
        ```
     Update openjdk path:
        ```ruby
        name "server-jre"
        default_version "1.8.0"
        raise "Server-jre can only be installed on x86_64 systems." unless _64_bit?
        
        whitelist_file "jre/bin/javaws"
        whitelist_file "jre/bin/policytool"
        whitelist_file "jre/lib"
        whitelist_file "jre/plugin"
        whitelist_file "jre/bin/appletviewer"
        
        version "1.8.0" do
            source path: "/usr/lib/jvm/java-8-openjdk-s390x/"
        end
        
        build do
            mkdir "#{install_dir}/embedded/jre"
            sync  "#{project_dir}/", "#{install_dir}/embedded/jre"
        end
        ```
    6. Update sqitch.rb
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/sqitch.rb config/software/.
        vi config/software/sqitch.rb
        ```
        Sqitch has updated a few times since the s3 caching version was stored and the original version is no longer available for download, so update the version and `md5` checksum
        
		From
		```ruby
		name "sqitch"
		default_version "0.973"
		```
		to
		```ruby
        name "sqitch"
        default_version "0.999"
        ```
		and from
		```ruby
		source url: "https://github.com/theory/#{name}/releases/download/v#{version}/app-sqitch-#{version}.tar.gz"
		```
        to
		```ruby
        source url: "http://www.cpan.org/authors/id/D/DW/DWHEELER/App-Sqitch-#{version}.tar.gz",
            md5: "b3a9cac1254e0e90e4cc09fc84a66c93"
        ```
    7. Create nodejs.rb from an empty file
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        vi config/software/nodejs.rb
        ```
        Nodejs doesn't support Linux on z Systems, however there is a fork of `nodejs` that does, adding the below as the content of the empty file edited above downloads that version of `nodejs`
        ```ruby
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
            command "#{install_dir}/embedded/bin/python ./configure --dest-cpu=s390x --prefix=#{install_dir}/embedded", env: env
            make "-j #{workers}", env: env
            make "install", env: env
        end
        ```
    8. Update openresty-lpeg.rb
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        vi config/software/openresty-lpeg.rb
        ```
        As we turned off `--with-luajit` earlier we need to alter the `openresty` include path, so update the make command to be the same as below
        
		From
		```ruby
		 env = with_standard_compiler_flags(with_embedded_path)

		if ppc64? || ppc64le?
			make "LUADIR=#{install_dir}/embedded/include", env: env
		else
			make "LUADIR=#{install_dir}/embedded/luajit/include/luajit-2.1", env: env
		end
		command "install -p -m 0755 lpeg.so #{install_dir}/embedded/lualib", env: env
		```
		to
		```ruby
        build do
            env = with_standard_compiler_flags(with_embedded_path)
            
            make "LUADIR=#{install_dir}/embedded/lua/include/", env: env
            command "install -p -m 0755 lpeg.so #{install_dir}/embedded/lualib", env: env
        end
        ```
    9. Update ohai.rb
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/ohai.rb config/software/.
        vi config/software/ohai.rb
        ```
        There is an issue with the available gems, the simplest solution is to convert the install to work within bundler - which guarantees that the relevant gems are in the correct location, so also update the file to match the below:
       
	   From 
		```ruby
		gem "build ohai.gemspec", env: env
		gem "install ohai*.gem" \
		" --no-ri --no-rdoc", env: env
		```
		to
		```ruby
        gem "build ohai.gemspec", env: env
        bundle "exec gem install ohai*.gem --no-ri --no-rdoc", env: env
        ```
        _**Note:** All we have done is perform the `gem install` inside of the `bundle exec`, this means that all the gems installed a few lines earlier with `bundle "install..` are available_
    10. Update opscode-chef-mover.rb
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        vi config/software/opscode-chef-mover.rb
        ```
        HiPE or High Performance Erlang as a requirement is a native compiler for Erlang, but isn't supported on Linux on z Systems yet. Removing the hipe requirement still works so we remove it from the `rebar.config`
        ```ruby
		
			  env['USE_SYSTEM_GECODE'] = "1"
			  env['REL_VERSION'] = "#{project.build_version}"
			  env['REBAR_PROFILE'] = profile_name
			  
			  command "cat rebar.config | grep -v hipe > rebar.config.mod", env: env
			  command "mv -f rebar.config.mod rebar.config", env: env
			  
			  make "omnibus", env: env

			  sync "#{project_dir}/_build/#{profile_name}/rel/mover/", "#{install_dir}/embedded/service/opscode-chef-mover/"
        ```
        _**Note:** We don't directly edit the `rebar.config` file as it is updated each time the build is run, but simply remove the `hipe` dependency at build time with the two additional `command` lines above the make statement_
    11. Update opscode-solr4.rb (Only for IBM Java)
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        vi files/private-chef-cookbooks/private-chef/recipes/opscode-solr4.rb
        ```
        Apache Solr relies on a non-standard option `-Xloggc` which doesn't exist on IBM's JDK, however there is an equivalent `-Xverbosegclog` so we replace it as below:
     
		From 
		```ruby
		if node['kernel']['machine'] == "x86_64"
			node.default['private_chef']['opscode-solr4']['command'] << " -Xloggc:#{File.join(solr_log_dir, "gclog.log")}"
		end
		```
		to
		```ruby
        if node['kernel']['machine'] == "x86_64"
			node.default['private_chef']['opscode-solr4']['command'] << " -Xverbosegclog:#{File.join(solr_log_dir, "gclog.log")}"
		end
        ```
        _**Note:** All we have done is replaced `-Xloggc` with `-Xverbosegclog`_

    12. Update ncurses.rb (RHEL/SLES **only**)
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/ncurses.rb config/software/.
        vi config/software/ncurses.rb
        ```
        The default version specifies some patch files to be installed, which are not available, nor really needed for this build, so comment out the code which includes them:
 
		From
		```ruby
		if version == "5.9"
			# Patch to add support for GCC 5, doesn't break previous versions
			patch source: "ncurses-5.9-gcc-5.patch", plevel: 1, env: env
		end
		```
		to
		```ruby
        #  if version == "5.9"
        #    # Patch to add support for GCC 5, doesn't break previous versions
        #    patch source: "ncurses-5.9-gcc-5.patch", plevel: 1
        #  end
        ```
        _**Note:** Comment out the whole if statement for version == "5.9"_
        
    13. Update health_check.rb
    
        ```shell
        vi $SRCRT/omnibus/lib/omnibus/health_check.rb
        ```
		
        Within the `health_check.rb` file update the `WHITELIST_LIBS` array adding the following at the end:
        ```ruby
        WHITELIST_LIBS = [
          .........
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
          /libhistory\.so/,
          /libtinfo\.so/
        ].freeze
        ```
        _**Note:** Don't include the `......`, it is just to indicate the end of the existing `WHITELIST_LIBS` array without making this document too large_
        
        If you were unable to work on a clean system you may have additional items that the healthcheck will show up - ideally this build would take place on a clean VM or a docker container so that no problems occur but if you see errors similar to:
        ```shell
                [HealthCheck] E | The following libraries have unsafe or unmet dependencies:
        --> /opt/opscode/embedded/lib/python2.7/lib-dynload/_sqlite3.so
        --> /opt/opscode/embedded/lib/python2.7/lib-dynload/_tkinter.so
        
                [HealthCheck] E | The following binaries have unsafe or unmet dependencies:
                [HealthCheck] E | The following libraries cannot be guaranteed to be on target systems:
        --> /usr/lib64/libsqlite3.so.0 (0x000003fffd0a5000)
        --> /usr/lib64/libtk8.6.so (0x000003fffd4ba000)
        --> /usr/lib64/libtcl8.6.so (0x000003fffd2db000)
        --> /usr/lib64/libXss.so.1 (0x000003fffcd88000)
        --> /lib64/libz.so.1 (0x000003fffcb61000)
        --> /usr/lib64/libpng16.so.16 (0x000003fffcb20000)
        ```
        Then you have additional libraries that have been incorporated into the build (most likely as part of the Ruby build that supports chef-server). At this point you have two choices, first you can ensure those libraries aren't present and start the build process again or secondly you can add the libraries to the whitelist.
        
        **WARNING: Adding libraries to the whitelist may cause problems when using the produced RPM - if this occurs either add the libraries to the machine before installing the RPM or rebuild the RPM without those libraries present**
        
        To add the libraries to the whitelist (and please ensure you've read the warning above) look for the output from the build process that looks like:
        ```shell
        --> /opt/opscode/embedded/lib/python2.7/lib-dynload/_tkinter.so
        DEPENDS ON: libXss.so.1
            COUNT: 1
            PROVIDED BY: /usr/lib64/libXss.so.1 (0x000003fffcd88000)
            FAILED BECAUSE: Unsafe dependency
        ```
        For each unique entry you need to add a line to the whitelist that matches the library that is missing, in the above example the line would be `/libXss\.so/,`, it must have a `/` at the beginning and end of the text and any `.` must be escaped `\.` - essentially it is a regex that matches against the line `libXss.so.1` in order to continue the build.
    14. Remove VERISIGN certificates
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/cacerts.rb config/software/.
        vi config/software/cacerts.rb
        ```
        Here all we need to comment out (add the `#`) all the lines at the end of the file from the below onwards
        ```ruby
        #VERISIGN_CERTS = <<-EOH
        #
        #Verisign Class 3 Public Primary Certification Authority
        #=======================================================
        #-----BEGIN CERTIFICATE-----
        ```
        _**Note:** These VERISIGN certificates are only added back in to access S3 cached items, which we can't use anyway (they were removed from the certificate package for security reasons)_
    
    15. Update chef.rb 
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/chef.rb config/software/.
        ```
        Only copy the file to the required folder, No modifications needed.

    16. Update libffi.rb (SLES 64bit **only**)

        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/libffi.rb config/software/.
        mkdir -p config/patches/libffi
        cp ../../omnibus-software/config/patches/libffi/libffi-3.2.1-disable-multi-os-directory.patch config/patches/libffi
        ```

        There is no need to modify the contents of the libffi.rb file, this is to update the gem version used by the build process. It is necessary to create a folder and copy across a patch used by this file.
    17. Update liblzma.rb

        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/liblzma.rb config/software/.
        ```
        There is no need to modify the contents of the liblzma.rb file, this is to update the gem version used by the build process.
    18. Update oc_erchef rebar dependencies

        A recent change to the erlang rebar dependencies has left them in an inconsistent state, and try to specify packages which are not available. Fix this by stepping back to earlier versions and specifying exact tags instead of master branch. There are three files to modify:

        ```shell
        cd $SRCRT/chef-server/omnibus/
        vi ../src/oc_erchef/apps/depsolver/rebar.config
		```
		Update the erlware_commons to be a tag, not master branch:

        From
		```
		{deps, [{erlware_commons, "",
			{git, "https://github.com/erlware/erlware_commons.git", {branch, "master"}}}]}.
		```
		to
        ```
        {deps, [{erlware_commons, "",
			{git, "https://github.com/erlware/erlware_commons.git", {tag, "v0.16.1"}}}]}.
        ```
        _**Note:** We have changed both `branch` to `tag` and `master` to `v0.16.1`_
        ```shell
        vi ../src/oc_erchef/rebar.config
        ```
        Update erlware_commons to be a tag, not master branch, and also add an explicit dependency for cf tag:

        From
		```
		{erlware_commons, "",
         {git, "https://github.com/erlware/erlware_commons", {branch, "master"}}},
		```
		to
		```
        {erlware_commons, "",
         {git, "https://github.com/erlware/erlware_commons", {tag, "v0.16.1"}}},
        {cf, ".*",
         {git, "https://github.com/project-fifo/cf", {tag, "0.1.2"}}},
        ```
        _**Note:** Again we changed `branch` to be `tag` and `master` to be `v0.16.1` and also added the `{cf,...` lines_
        ```shell
        vi ../src/oc_erchef/rebar.lock
        ```
        ```
		
			 {<<"cf">>,
			  {git,"https://github.com/project-fifo/cf",
					{ref,"16dcc2b2a0817b006df4fb90f6f0fbe973f90a51"}},
			  0},
  
        ```
        _**Note:** Add the `cf` 4 lines just after bcrypt._


    19. Update openssl.rb (SLES 11 **only**)
    
        SLES 11 has openssl 0.9.8 available by default and recent changes (relating to security standards) have caused www.openssl.org to modify their ssl handling, this means that openssl 0.9.8 cannot easily be used to connect to openssl.org, we fix this by using the `http` url rather than the `https` - this isn't a problem for the build process as we have previously supplied the MD5 hash to prevent corruption / security issues when downloading.
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/openssl.rb config/software/.
        vi config/software/openssl.rb
        ```
        Now change the URL so that it uses `http`:

        From
		```ruby
		source url: "https://www.openssl.org/source/openssl-#{version}.tar.gz", extract: :lax_tar
		```
        to
		```ruby
		source url: "http://www.openssl.org/source/openssl-#{version}.tar.gz" 
		```

        _**Note:** We do not change the `source md5:` entry for version `1.0.1p` so that we know the version we've downloaded is the version we were expecting_

    20. Add chef_objects to the applications in oc_chef_authz.app.src
        
        To avoid chef_object from being undefined we need to compile oc_chef_authz before chef_objects by doing the following:
		```
		cd $SRCRT/chef-server/
		vi src/oc_erchef/apps/oc_chef_authz/src/oc_chef_authz.app.src
		```
		
		Add chef_objects to the applications as mentioned below:
		```
		{applications, [kernel,
				  stdlib,
				  stats_hero,
				  chef_objects,
				  sqerl]},
		{mod, {oc_chef_authz_app, []}}
        ```
       
    21. On Ubuntu 16.04 with gcc version 5.x:
        Building chef server on Ubuntu with gcc version 5.x requires an option '-fwrapv' to be added for few modules. 

         a. Update opscode-chef-mover.rb 
        ```shell
         cd $SRCRT/chef-server/omnibus/
         vi config/software/opscode-chef-mover.rb
        ```
 
        ```ruby
          build do
          env = with_standard_compiler_flags(with_embedded_path)
          env['CPPFLAGS'] << " -fwrapv "
          command "cat rebar.config | grep -v hipe > rebar.config.mod", env: env
        ```

        b.oc_erchef.rb 
        
        ```shell
         cd $SRCRT/chef-server/omnibus/
         vi config/software/oc_erchef.rb
        ```
      
        ```ruby
          build do
          env = with_standard_compiler_flags(with_embedded_path)
          env['CPPFLAGS'] << " -fwrapv "
          env['USE_SYSTEM_GECODE'] = "1"
        ```
 

11 . Install or stub fakeroot and build Chef Server Omnibus

    You may already have fakeroot available on your system, but if not (and as we are building as root) you can use the following to temporarily stub fakeroot:
    ```shell
    echo '"$@"' > /usr/bin/fakeroot
    chmod +x /usr/bin/fakeroot
    ```
    _**Note:**_ 
	
    _i) Don't forget to remove the fakeroot script after the build has completed_

    _ii) Remove gccgo path from the PATH variable only for SLES11 and RHEL6 before starting the build as it causes problems while building ncurses module._

    ```shell
    cd $SRCRT/chef-server/omnibus/
    bin/omnibus build chef-server
    ```

    _**Note:**_  

    _i) Occasionally during the download phase there will be errors similar to `===> ...net_fetcher.rb:180:in 'each': comparison of NilClass with 861 failed (ArgumentError)...`, if you see these, start the build process again and it should download the second time._  

    _ii) If you get the following error `===> cc1plus: warnings being treated as errors
c_src/double-conversion/utils.h: In function 'void double_conversion::CutToMaxSignificantDigits(double_conversion::Vector<const char>, int, char*, int*)':
c_src/double-conversion/utils.h:195: error: assuming signed overflow does not occur when assuming that (X - c) >= X is always false`, then_

    _Add the following statement to the config files of the failed modules (for e.g. oc_bifrost, oc_erchef, opscode-chef-mover etc.) and start the build process again and it should build the second time._
    ```shell
    cd $SRCRT/chef-server/omnibus/config/software
    vi oc_erchef.rb
    ```
    _Add the below line to the file._
    ```shell
    env['CPPFLAGS'] << " -fno-strict-overflow "
    ```

    _iii) If you get the following error `===> /usr/local/lib/ruby/2.2.0/net/http.rb:923:in `connect': SSL_connect returned=1 errno=0 state=error: certificate verify failed (OpenSSL::SSL::SSLError)`, then_
      
      _Download the following certificate, set environment variable and rerun the build:_
     ```shell
     cd /<source_root>/
     wget http://curl.haxx.se/ca/cacert.pem --no-check-certificate
     export SSL_CERT_FILE=/<source_root>/cacert.pem
     ```
    
	_iv) If you get the following error `An error occurred while installing eventmachine (0.12.10), and Bundler cannot continue. Make sure that `gem install eventmachine -v '0.12.10'` succeeds before bundling.`, then_
	 _Change the version of the eventmachine gem to 1.2.0.1 and run `bundle install`_
	 ```shell
	 cd ../src/opscode-expander
	 vi Gemfile
	 ```
	 _Change the version of `eventmachine` to `1.2.0.1`_
	 ```ruby
	 gem "eventmachine", '~> 1.2.0.1'
	 ```
	 _Remove the Gemfile.lock and install the Gems_
	 ```shell
	 rm Gemfile.lock
	 bundle install
	 ```
	 
    **Once this completes your built RPM will be in the `pkg` subdirectory.**  
    
    If you wanted to clean your build process and start again you can do some of the following:
    ```shell
    bin/omnibus clean chef-server
    # or #
    bin/omnibus clean chef-server --purge
    ```
    These will clean the build tree in the first case and purge the downloaded files in the second (causing it to download everything). Removing the git cache is a little more direct:
    ```shell
    rm -rf /var/cache/omnibus/cache/git_cache/
    ```
    Which will cause the build process to rebuild everything even if nothing has changed.  

12 . **Optionally** Install and test the resulting RPM

    First you will need to move the RPM to the destination machine - if you intend to install the RPM on the machine that also built it, please be aware that part of the build tree sits in the same location that it will install to and the results will be unpredictable
    ```shell
    rpm -ivh <name_of_rpm>
    chef-server-ctl reconfigure
    chef-server-ctl test
    ```
    This will install the rpm, then reconfigure it for default settings and finally run the tests - when the tests run 2 are expected to fail / pend as they are known bugs (at the time of writing this guide).  
    Uninstalling is as simple as following the below instructions:
    ```shell
    chef-server-ctl cleanse
    rpm -qa | grep chef
    rpm -e <chef_rpm_name>
    ```
    _**Note:** Where `<chef_rpm_name>` is the response from the `rpm -qa | gre..` line  (alternately you can use `rpm -qa | grep chef | xargs rpm -e` but be careful as this will uninstall anything containing chef). Finally after uninstalling check the `/opt/opscode` directory as some items may be left behind._

   On Ubuntu 16.04: 

    Install and test using:
    ```shell
    dpkg -i <name_of_chef_pkg>
    chef-server-ctl reconfigure
    chef-server-ctl test
    ```
    Uninstall as:
    ```shell
    chef-server-ctl cleanse
    dpkg -l | grep 'chef'
    dpkg --purge <name_of_chef_pkg>
    ```
