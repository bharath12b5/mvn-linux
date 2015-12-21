The **Chef Server Omnibus** RPM can be built for Linux on z Systems running RHEL 7.1/6.6 or SLES 12/11 by following these instructions.  Version 12.1.2 has been successfully built & tested this way.

_**General Notes:**_ 	
_i) When following the steps below please use a superuser / root. This isn't best practise (but the build process is much more stable as a superuser) however this RPM could be built on a VM / Container so that root isn't exposed._  
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it. During the build the environment variable `$SRCRT` will be set to hold this and make the recipe easier to follow_  
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
    You may already have some of these packages installed - install any that are missing if needed.  
    _**Note:** The `tcl-devel`, `tk-devel` and `gdbm-devel` packages should NOT be installed (otherwise the Ruby build process will incorporate them and Chef Server Omnibus may fail with a healthcheck dependency error later)_
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
    
    Once you have completed the Ruby installation continue to follow the process below
4. Correct the gem environment and install bundler

    ```shell
    export GEM_HOME=<USER_HOME_DIR>/.gem/ruby
    export PATH=$GEM_HOME/bin:$PATH
    gem install bundler
    ```
    _Where `<USER_HOME_DIR>` is the home directory of the user you are installing under._  
    _**Note:** Run `gem env` to verify the state of the environment, if later on you have issues installing / running ruby gems please ensure the environment is set correctly._
5. Setup `<source_root>` and `$SRCRT`

    ```shell
    mkdir /<source_root>/
    cd /<source_root>
    export SRCRT=`pwd`
    ```
    _**Note:** You can use an existing directory for `<source_root>` but this might make clean-up a little more awkward afterwards_
6. Download the build process code

    ```shell
    cd $SRCRT
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
    cd $SRCRT/chef-server/
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
        cd $SRCRT/chef-server/omnibus/
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
        cd $SRCRT/chef-server/omnibus/
        vi config/projects/chef-server.rb
        ```
        Here we need to remove an override and update the version of Ruby bundled - this is because the s3 cache contains an old version of the cacerts file which is no longer available outside of s3 caching so use the latest instead, and the `2.1.4` version of Ruby has some SSL issues:
        ```ruby
        #override :cacerts, version: '2014.08.20'
        override :rebar, version: "2.0.0"
        override :berkshelf2, version: "2.0.18"
        override :rabbitmq, version: "3.3.4"
        override :erlang, version: "17.5"
        override :ruby, version: "2.1.6"
        override :chef, version: "9a3e6e04f3bb39c2b2f5749719f0c21dd3f3f2ec"
        ```
        _**Note:**  Remove the `:cacerts` version line by commenting it out, and change the `:ruby` version to be `2.1.6`_
    3. Update openresty.rb
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/openresty.rb config/software/.
        vi config/software/openresty.rb
        ```
        Copy the default `openresty.rb` file in order to be able to make the changes otherwise the build process will download a fresh copy.
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
        cd $SRCRT/chef-server/omnibus/
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
        cd $SRCRT/chef-server/omnibus/
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
        cd $SRCRT/chef-server/omnibus/
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
        Firstly the version of ohai available as `master` (at the time of writing) has a conflicting gem requirement, to fix this we specify the version to be `8.5.1`, so update it as below:
        ```ruby
        name "ohai"
        default_version "8.5.1"
        
        source git: "git://github.com/opscode/ohai"
        ```
        Secondly there is an issue with the available gems, the simplest solution is to convert the install to work within bundler - which guarantees that the relevant gems are in the correct location, so also update the file to match the below:
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
        cd $SRCRT/chef-server/omnibus/
        vi config/software/opscode-chef-mover.rb
        ```
        HiPE or High Performance Erlang as a requirement is a native compiler for Erlang, but isn't supported on Linux on z Systems yet. Removing the hipe requirement still works so we remove it from the `relx.config`
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
        cd $SRCRT/chef-server/omnibus/
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
        cd $SRCRT/chef-server/omnibus/
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
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/ncurses.rb config/software/.
        vi config/software/ncurses.rb
        ```
        The default version specifies some patch files to be installed, which are not available, nor really needed for this build, so comment out the code which includes them:
        ```ruby
        #  if version == "5.9"
        #    # Update config.guess to support platforms made after 2010 (like aarch64)
        #    patch source: "config_guess_2015-09-24.patch", plevel: 0
        #
        #    # Patch to add support for GCC 5, doesn't break previous versions
        #    patch source: "ncurses-5.9-gcc-5.patch", plevel: 1
        #  end
        ```
        **Note:** Comment out the whole if statement for version == "5.9"
    13. Update health_check.rb
    
        For RHEL platforms the health_check.rb list is installed as a gem so the health_check.rb file would be found via the command below:
        ```shell
        find $GEM_HOME -name health_check.rb
        ```
        For SLES platforms we manually extracted the omnibus gem so it will be available in:
        ```shell
        vi $SRCRT/omnibus/lib/omnibus/health_check.rb
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
        Then you have additional libraries that have been incorporated into the build (most likely as part of the Ruby build that supports chef-server). At this point you have two choices, first you can ensure those libraries aren't present and start the build prpocess again or secondly you can add the libraries to the whitelist.
        
        **WARNING: Adding libraries to the whitelist may cause problems when using the produced RPM - if this occurs either add the libraries to the machine before installing the RPM or rebuild the RPM without those libraries present**
        
        To add the libraries to the whitelist (and please ensure you've read the warning above) look for the output from the build process that looks like:
        ```shell
        --> /opt/opscode/embedded/lib/python2.7/lib-dynload/_tkinter.so
        DEPENDS ON: libXss.so.1
            COUNT: 1
            PROVIDED BY: /usr/lib64/libXss.so.1 (0x000003fffcd88000)
            FAILED BECAUSE: Unsafe dependency
        ```
        For each unique entry you need to add a line to the whitelist that matches the library that is missing, in the above example the line would be `/libXss\.so/,`, it must have a `/` at the beginning and end of the text and any `.` must be escaped `\.` - essentially it is a regex that matches against the the line `libXss.so.1` in order to continue the build.
    15. Remove VERISIGN certificates
    
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/cacerts.rb config/software/.
        vi config/software/cacerts.rb
        ```
        Here all we need to do is comment out two sections, one that adds some certificates and the other that defines the certificates to add:
        ```ruby
        build do
          mkdir "#{install_dir}/embedded/ssl/certs"
        
        # Append the 1024bit Verisign certs so that S3 continues to work
        #  block do
        #    unless File.foreach("#{project_dir}/cacert.pem").grep(/^Verisign Class 3 Public Primary Certification Authority$/).any?
        #      File.open("#{project_dir}/cacert.pem", "a") { |fd| fd.write(VERISIGN_CERTS) }
        #    end
        #  end
        
        copy "#{project_dir}/cacert.pem", "#{install_dir}/embedded/ssl/certs/cacert.pem"
        ```
        And also comment out (add the `#`) all the lines at the end of the file from the below onwards
        ```ruby
        #VERISIGN_CERTS = <<-EOH
        #
        #Verisign Class 3 Public Primary Certification Authority
        #=======================================================
        #-----BEGIN CERTIFICATE-----
        ```
        _**Note:** These VERISIGN certificates are only added back in to access S3 cached items, which we can't use anyway (they were removed from the certificate package for security reasons)_
    16. Update chef.rb to handle net-ssh version conflict
    
        Similarly to the Ohai issue seen earlier we have to ensure the correct version of the gem is loaded, this is complicated by what appears to be an odd bug in bundler than prevents it resolving it automatically.
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/chef.rb config/software/.
        vi config/software/chef.rb
        ```
        Update the file to have **four** entries in it:
        ```ruby
        # install the whole bundle first
        command "sed -i '/source/a gem \"net-ssh\", \"~> 2.6\"' Gemfile", env: env
        command "sed -i '/source/a gem \"net-ssh\", \"~> 2.6\"' Gemfile", env: env
        command "sed -i '/source/a gem \"net-ssh\", \"~> 2.6\"' Gemfile", env: env
        command "sed -i '/source/a gem \"net-ssh\", \"~> 2.6\"' Gemfile", env: env
        bundle "install --without server docgen", env: env
        ```
        _**Note:** Four (yes, really) additional `command...` lines are necessary as otherwise bundler does not handle the `~> 2.6` requirement before the conflicting requirement otherwise (this appears to be an issue in bundler dependency resolution order)_
    17. Update libffi.rb (SLES 64bit **only**)

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
    17. Update oc_erchef rebar dependencies

        A recent change to the erlang rebar dependencies has left them in an inconsistent state, and try to specify packages which are not available. Fix this by stepping back to earlier versions and specifying exact tags instead of master branch. There are three files to modify:

        ```shell
        cd $SRCRT/chef-server/omnibus/
        vi ../src/oc_erchef/apps/depsolver/rebar.config
        ```
        Update the erlware_commons to be a tag, not master branch:
        ```ruby
        {erl_opts, [debug_info, warnings_as_errors]}.

        {deps, [{erlware_commons, "",
                 {git, "https://github.com/erlware/erlware_commons.git", {tag, "v0.16.1"}}}]}.

        {cover_enabled, true}.
        ```
        _**Note:** We have changed both `branch` to `tag` and `master` to `v0.16.1`_
        ```shell
        vi ../src/oc_erchef/rebar.config
        ```
        Update erlware_commons to be a tag, not master branch, and also add an explicit dependency for cf tag:
        ```ruby
        {erlware_commons, "",
         {git, "https://github.com/erlware/erlware_commons", {tag, "v0.16.1"}}},
        {cf, ".*",
         {git, "https://github.com/project-fifo/cf", {tag, "0.1.2"}}},
        ```
        _**Note:** Again we changed `branch` to be `tag` and `master` to be `v0.16.1` but we also added the `{cf,...` lines_
        ```shell
        vi ../src/oc_erchef/rebar.config.lock
        ```
        Update corresponding explicit git SHA-1 ids for erlware_commons and add in entries for cf, as this will in fact be used and 'override' the previous tags:
        ```ruby
        {erlware_commons,".*",
                        {git,"https://github.com/erlware/erlware_commons.git",
                             "95a8e3c32d107e6d8909fae4689b11fb1cd20857"}},
        {cf,".*",
                        {git,"https://github.com/project-fifo/cf",
                             "16dcc2b2a0817b006df4fb90f6f0fbe973f90a51"}},
        ```
        _**Note:** Update the SHA-1 id for the `erlware_commons` entry and then add the `cf` 3 lines just after._

    18. Update openssl.rb (SLES 11 **only**)
    
        SLES 11 has openssl 0.9.8 available by default and recent changes (relating to security standards) have caused www.openssl.org to modify their ssl handling, this means that openssl 0.9.8 cannot easily be used to connect to openssl.org, we fix this by using the `http` url rather than the `https` - this isn't a problem for the build process as we have previously supplied the MD5 hash to prevent corruption / security issues when downloading.
        ```shell
        cd $SRCRT/chef-server/omnibus/
        cp ../../omnibus-software/config/software/openssl.rb config/software/.
        vi config/software/openssl.rb
        ```
        Now change the URL so that it uses `http`:
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
    cd $SRCRT/chef-server/omnibus/
    bin/omnibus build chef-server
    ```
    _**Note:** Occasionally during the download phase there will be errors similar to `...net_fetcher.rb:180:in 'each': comparison of NilClass with 861 failed (ArgumentError)...`, if you see these, start the build process again and it should download the second time._  
    
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
    _**Note:** Where `<chef_rpm_name>` is the response from the `rpm -qa | gre..` line  (alternately you can use `rpm -qa | grep chef | xargs rpm -e` but be careful as this will uninstall anything containing chef). Finally after uninstalling check the `/opt/opscode` directory as some items may be left behindChef Server 12.1.2 on RHEL 7.1_6.6 or SLES 12_11.md