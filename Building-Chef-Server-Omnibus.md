The **Chef Server Omnibus** RPM can be built for Linux on z Systems running RHEL 7.1/6.6 or SLES 12/11 by following these instructions.  Version 12.1.2 has been successfully built & tested this way.

_**General Notes:**_ 	
_i) When following the steps below please use a superuser / root. This isn't best practise (but the build process is much more stable as a superuser) however this RPM could be built on a VM / Container so that root isn't exposed._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it_  
_iii) For convenience `vi` has been used in the instructions below when editing files, replace with your desired editing program if required_

## Building Chef Server Omnibus

1. Install the build time dependencies

    On RHEL 7.1 systems
    ```shell
    yum install git ruby-devel rubygems make gcc gcc-c++ patch readline-devel bzip2 tar java-1.7.1-ibm-devel libxml2-devel which wget curl rpm-build
    ```
    On RHEL 6.6 systems:
    ```shell
    yum install git make gcc gcc-c++ bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel ncurses-devel gcc make wget tar patch bzip2 java-1.7.1-ibm-devel which libxml2-devel curl rpm-build
    ```
    On SLES 12 systems
    ```shell
    zypper install git make gcc tar gcc-c++ patch readline-devel bzip2 tar java-1_7_1-ibm-devel which wget libxml2-devel curl timezone rpm-build libopenssl-devel bison flex libyaml-devel libffi48-devel zlib-devel ncurses-devel
    ```
    On SLES 11 systems
    ```shell
    zypper install git make gcc47 tar gcc47-c++ bison flex libopenssl-devel libyaml-devel libffi-devel readline-devel zlib-devel ncurses-devel make wget tar autoconf patch bzip2 java-1_7_0-ibm-devel libxml2-devel curl timezone
    ```
    You may already have some of these packages installed - just install any that are missing.  
    _**Note:** The `tcl-devel`, `tk-devel` and `gdbm-devel` packages should NOT be installed (otherwise the Ruby build process will incorporate them and Chef Server Omnibus may fail with a Healthcheck dependency error later)_
2. Correct the gcc linking for SLES 11 **only**

    ```shell
    ln -s /usr/bin/cpp-4.7 /usr/bin/cpp
    ln -s /usr/bin/g++-4.7 /usr/bin/g++
    ln -s /usr/bin/gcc-4.7 /usr/bin/gcc
    ln -s /usr/bin/gcc-ar-4.7 /usr/bin/gcc-ar
    ln -s /usr/bin/gcc-nm-4.7 /usr/bin/gcc-nm
    ln -s /usr/bin/gcc-ranlib-4.7 /usr/bin/gcc-ranlib
    ln -s /usr/bin/gcc /usr/bin/cc
    ln -s /usr/bin/g++ /usr/bin/c++
    ```
    _**Note:** This is necessary because the standard gcc version available on SLES 11 is 4.3, and chef requires 4.4+ to build, but the 4.7 gcc package available on SLES 11 suffixes all the executables_
3. Build Ruby from source (**only for RHEL 6.6 / SLES 11 / SLES 12**)

    **WARNING:** Do not perform the Ruby install dependencies step as that specifies the tcl / tk / gdbm libraries. The dependencies above cover the Ruby build process. If you already have tcl-devel, tk-devel or gdbm-devel installed you'll need to remove them for the duration of this build
    
    The Ruby version available on the RHEL 6.6 and SLES 11 package repositories is too low level and the Ruby version available on SLES 12 has some configuration issues so build and install Ruby yourself following the instructions [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby).
    
    Once you have completed the Ruby install continue to follow the process below
4. Correct the gem environment and install bundler

    ```shell
    export GEM_HOME=/home/<USER>/.gem/ruby
    export PATH=$GEM_HOME/bin:$PATH
    gem install bundler
    ```
    _Where `<USER>` is the user you are installing under - unless your home directory is elsewhere._  
    _**Note:** Run `gem env` to verify the state of the environment, if later on you have issues installing / running ruby gems please ensure the environment is set correctly._
5. Download the build process code

    ```shell
    cd /<source_root>/
    git clone --depth 100 https://github.com/chef/omnibus-software
    git clone --depth 100 https://github.com/chef/chef-server
    ```
    _**Note:** The `--depth 100` flag is used to reduce the amount downloaded, in the future this might mean that specific checkouts won't work (as the total number of commits has moved on), in that case either increase the value or omit the tag_  
    **Only** the SLES platforms require one additional repository (and one modification)
    ```shell
    git clone --depth 100 https://github.com/chef/omnibus
    vi omnibus/lib/omnibus/packager.rb
    ```
    Add update the packaging list to understand SLES platforms as follows:
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
    }.freeze
    ```
    _**Note:** `'suse'` was added and pointing to RPM to support SLES platforms_  
    This is because the omnibus gem doesn't support / recognise `'suse'` as a platform and so drops into a `makeself` package.
6. Checkout the required version of Chef Server and download the Ruby Gem requirements

    ```shell
    cd chef-server/
    git checkout 12.1.2
    cd omnibus
    vi Gemfile
    ```
    Update the Gemfile to add a couple of missing Gems (and on SLES only point omnibus to the git repo downloaded earlier)
    ```ruby
    gem 'omnibus', path: '/<source_root>/omnibus'
    gem 'omnibus-software', github: 'opscode/omnibus-software'
    gem 'rake'
    gem 'json'
    gem 'json_pure'
    ```
    _**Note:** Add the `json` and `json_pure` gems on all platforms, but *ONLY* change the `gem 'omnibus'` line on SLES platforms_  
    
    Finally install the build time Gems
    ```shell
    bundle install --binstubs
    git config --global user.email "you@example.com"
    ```
    _**Note:** Git needs to be configured with at least a user.email (if this is already set then there is no need to update it) in order for the git caching to work - the caching is useful to recover from missing any steps without having to do a complete rebuild. If you'd prefer not to have to configure git you can disable git caching in **7.i**_
7. Update a number of different files

    1. Update omnibus.rb
    
        ```shell
        vi omnibus.rb
        ```
        Turn off s3 caching (we need to build a number of items differently from the s3 cached version), and optionally turn off git caching (it is recommended to keep git caching on).
        ```ruby
        # Disable git caching
        # ------------------------------
        # use_git_caching false
        
        # Enable S3 asset caching
        # ------------------------------
        use_s3_caching false
        s3_access_key  ENV['AWS_ACCESS_KEY_ID']
        s3_secret_key  ENV['AWS_SECRET_ACCESS_KEY']
        s3_bucket      'opscode-omnibus-cache'
        ```
        _**Note:** Uncomment (remove the `#`) from the `# use_git_caching false` line to disable git caching_
    2. Update chef-server.rb
    
        ```shell
        vi config/projects/chef-server.rb
        ```
        Here we need to remove an override and update the version of Ruby bundled - this is because the s3 cache contains an old version of the cacerts file which is no longer available outside of s3 caching so just use the latest instead, and the `2.1.4` version of Ruby has some SSL issues:
        ```ruby
        #override :cacerts, version: '2014.08.20'
        override :rebar, version: "2.0.0"
        override :berkshelf2, version: "2.0.18"
        override :rabbitmq, version: "3.3.4"
        override :erlang, version: "17.5"
        override :ruby, version: "2.1.6"
        override :chef, version: "9a3e6e04f3bb39c2b2f5749719f0c21dd3f3f2ec"
        ```
        _**Note:** Just remove the `:cacerts` version line by commenting it out, and change the `:ruby` version to be `2.1.6`_
    3. Update openresty.rb
    
        ```shell
        cp ../../omnibus-software/config/software/openresty.rb config/software/.
        vi config/software/openresty.rb
        ```
        We had to copy the default `openresty.rb` file in order to be able to make the changes otherwise the build process will download a fresh copy.
        ```ruby
            # Options inspired by the OpenResty Cookbook
            #'--with-md5-asm',
            #'--with-sha1-asm',
            #'--with-pcre-jit',
            '--with-lua51',
            '--without-http_ssi_module',
        ```
        _**Note:** Comment out the 3 `asm` and `jit` modules as they are not supported, but you have to change the `--with-luajit` option to `--with-lua51` as the default for lua (if not specified) is now with jit which again is not supported._
    4. Update libossp-uuid.rb
    
        ```shell
        vi config/software/libossp-uuid.rb
        ```
        The original source is no longer available, so change the url as below:
        ```ruby
        source url: "https://gnome-build-stage-1.googlecode.com/files/uuid-1.6.2.tar.gz",
            md5: "5db0d43a9022a6ebbbc25337ae28942f"
        ```
        _**Note:** The `md5` checksum should not change as it is the same file just from a different location_
    5. Update server-jre.rb
    
        ```shell
        cp ../../omnibus-software/config/software/server-jre.rb config/software/.
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
    6. Update sqitch.rb
    
        ```shell
        cp ../../omnibus-software/config/software/sqitch.rb config/software/.
        vi config/software/sqitch.rb
        ```
        Sqitch has updated a few times since the s3 caching version was stored and the original version is no longer available for download, so update the version and `md5` checksum
        ```ruby
        name "sqitch"
        default_version "0.999"
        
        dependency "perl"
        dependency "cpanminus"
        
        source url: "http://www.cpan.org/authors/id/D/DW/DWHEELER/App-Sqitch-#{version}.tar.gz",
            md5: "b3a9cac1254e0e90e4cc09fc84a66c93"
        ```
    7. Create nodejs.rb
    
        ```shell
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
        vi config/software/openresty-lpeg.rb
        ```
        As we turned off `--with-luajit` earlier we need to alter the `openresty` include path, so update the make command to be the same as below
        ```ruby
        build do
            env = with_standard_compiler_flags(with_embedded_path)
            
            make "LUADIR=#{install_dir}/embedded/lua/include/", env: env
            command "install -p -m 0755 lpeg.so #{install_dir}/embedded/lualib", env: env
        end
        ```
    9. Update ohai.rb
    
        ```shell
        cp ../../omnibus-software/config/software/ohai.rb config/software/.
        vi config/software/ohai.rb
        ```
        Here there is an issue with the available gems, the simplest solution to convert the install to work within bundler - which guarantees that the relevant gems are in the correct location, so update the file to match the below:
        ```ruby
        build do
            env = with_standard_compiler_flags(with_embedded_path)
            
            bundle "install --without development", env: env
            
            gem "build ohai.gemspec", env: env
            bundle "exec gem install ohai*.gem --no-ri --no-rdoc", env: env
        end
        ```
        _**Note:** All we have done is perform the `gem install` inside of the `bundle exec`, this means that all the gems installed a few lines earlier with `bundle "install..` are available_
    10. Update opscode-chef-mover.rb
    
        ```shell
        vi config/software/opscode-chef-mover.rb
        ```
        HiPE or High Performance Erlang is a native compiler for Erlang, but isn't supported on Linux on z Systems but has unfortunately been added as a requirement for chef-mover. Removing the hipe requirement still works so we remove it from the `relx.config`
        ```ruby
        build do
            env = with_standard_compiler_flags(with_embedded_path)
            
            make "distclean", env: env
            command "cat relx.config | grep -v hipe > relx.config.mod", env: env
            command "mv -f relx.config.mod relx.config", env: env
            make "rel", env: env
        ```
        _**Note:** We don't directly edit the `relx.config` file as it is updated each time the build is run, but simply remove the `hipe` dependency at build time with the two additional `command` lines above_
    11. Update opscode-solr4.rb
    
        ```shell
        vi files/private-chef-cookbooks/private-chef/recipes/opscode-solr4.rb
        ```
        Apache Solr is relies on a non-standard option `-Xloggc` which doesn't exist on IBM's JDK, however there is an equivalent `-Xverbosegclog` so we replace it as below:
        ```ruby
        # Enable GC Logging (very useful for debugging issues)
        node.default['private_chef']['opscode-solr4']['command'] << " -Xverbosegclog:#{File.join(solr_log_dir, "gclog.log")} -verbose:gc -XX:+PrintHeapAtGC -XX:+PrintGCTimeStamps -XX:+PrintGCDetails -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintTenuringDistribution"
        ```
        _**Note:** All we have done is replaced `-Xloggc` with `-Xverbosegclog`_
    12. Update old_postgres_cleanup.rb
    
        ```shell
        vi files/private-chef-cookbooks/private-chef/recipes/old_postgres_cleanup.rb
        ```
        `postgres` has changed name between chef versions to `postgresql`, however the cleanup script doesn't correctly handle a new installation situation so we add a simple check to prevent errors during installation
        ```ruby
        runit_service "postgres" do
            action [:stop, :disable]
            not_if { not File.exist?('/opt/opscode/service/postgres') }
        end
        ```
        _**Note:** The issue is only when attempting to stop a non-existant `postgres` service, so we only protect that `runit_service` call_
    13. Update ncurses.rb
    
        ```shell
        cp ../../omnibus-software/config/software/ncurses.rb config/software/.
        vi config/software/ncurses.rb
        ```
        Like earlier items the download site for ncurses no longer provides this specific version, to fix this simply change the default version to be downloaded as below:
        ```ruby
        name "ncurses"
        default_version "5.9"
        
        dependency "libtool" if aix?
        dependency "patch" if solaris2?
        ```
        _**Note:** The only change is to the `default_version` in order to download `5.9` rather than `5.9-20150530`_
    13. Update health_check.rb
    
        For RHEL platforms the health_check.rb list is installed as a gem so the health_check.rb file would be found via the command below:
        ```shell
        find $GEM_HOME -name health_check.rb
        ```
        For SLES platforms we manually extracted the omnibus gem so it will be available in:
        ```shell
        vi /<source_root>/omnibus/lib/omnibus/health_check.rb
        ```
        _**Note:** For RHEL platforms `vi` the result of the `find $GEM_HOME...` command_  
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
    15. Update libffi.rb (SLES 64bit **only**)
    
        ```shell
        cp ../../omnibus-software/config/software/libffi.rb config/software/.
        ```
        There is no need to modify the contents of the libffi.rb file as it has already been updated to support SLES, however the gem version used by the build process has not yet been updated.
    16. Update openssl.rb (SLES 11 **only**)
    
        SLES 11 has openssl 0.9.8 available by default and recent changes (relating to security standards) have caused www.openssl.org to modify their ssl handling, this means that openssl 0.9.8 cannot easily be used to connect to openssl.org, we fix this by using the `http` url rather than the `https` - this isn't a problem for the build process as we have previously supplied the MD5 hash to prevent corruption / security issues when downloading.
        ```shell
        cp ../../omnibus-software/config/software/openssl.rb config/software/.
        vi config/software/openssl.rb
        ```
        Now just change the URL so that it uses `http`:
        ```ruby
        default_version "1.0.1p"
        
        source url: "http://www.openssl.org/source/openssl-#{version}.tar.gz" 
        
        version("1.0.1m") { source md5: "d143d1555d842a069cb7cc34ba745a06" }
        version("1.0.1p") { source md5: "7563e92327199e0067ccd0f79f436976" }
        ```
        _**Note:** We do not change the `source md5:` entry for version `1.0.1p` so that we know the version we've downloaded is the version we were expecting_

8. Install or stub fakeroot and build Chef Server Omnibus

    You may already have fakeroot available on your system, but if not (and as we are building as root) you can use the following to temporarily stub fakeroot:
    ```shell
    echo '"$@"' > /usr/bin/fakeroot
    chmod +x /usr/bin/fakeroot
    ```
    _**Note:** Don't forget to remove the fakeroot script after the build has completed_
    ```shell
    bin/omnibus build chef-server
    ```
    _**Note:** Occasionally during the download phase there will be errors similar to `...net_fetcher.rb:180:in 'each': comparison of NilClass with 861 failed (ArgumentError)...`, if you see these just start the build process again and it should download the second time._  
    
    Once this completes your built RPM will be in the `pkg` subdirectory.  
    
    If you wanted to clean your build process and start again you can do some of the following:
    ```shell
    bin/omnibus clean chef-server
    # or #
    bin/omnibus clean chef-server --purge
    ```
    These will clean the build tree in the first case and purge the downloaded files in the second (causing it to redownload everything). Removing the git cache is a little more direct:
    ```shell
    rm -rf /var/cache/omnibus/cache/git_cache/
    ```
    Which will cause the build process to rebuild everything even if nothing has changed.
9. **Optionally** Install and test the resulting RPM

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
    _**Note:** Where `<chef_rpm_name>` is the response from the `rpm -qa | gre..` line  (alternately you can use `rpm -qa | grep chef | xargs rpm -e` but be careful as this will uninstall anything containing chef). Finally after uninstalling check the `/opt/opscode` directory as some items may be left behind_