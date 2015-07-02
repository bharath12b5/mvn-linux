The Chef Client Omnibus version 12.0.4 RPM can be built for Linux on z Systems running RHEL 7.1/6.6 & SLES 12/11 by following these instructions. (Chef is available at https://www.chef.io/, the github repository for the client can be found at https://github.com/chef/chef and the Omnibus build process at https://github.com/chef/omnibus and https://github.com/chef/omnibus-software):

_**NOTE:** When following the steps below please use a superuser / root. This isn't best practise (the build process is much more stable as a superuser) but this RPM could be built on a VM / Container so that root isn't exposed._

1. Install the necessary dependencies (these include the Ruby build dependencies for step 2)

    For **RHEL 7.1 & 6.6** use the following
    ```shell
    yum install bison flex openssl-devel libyaml-devel libffi48-devel readline-devel zlib-devel ncurses-devel sqlite-devel gcc make wget tar gcc-c++ rpm-build unzip svn git patch
    ```
    For **SLES 12** (note the package name differences)
    ```shell
    zypper install bison flex libopenssl-devel libyaml-devel libffi48-devel readline-devel zlib-devel ncurses-devel sqlite3-devel gcc make wget tar gcc-c++ rpm-build unzip subversion git patch
    ```
    And finally for **SLES 11** use the following _(only difference from SLES 12 is libffi)_
    ```shell
    zypper install bison flex libopenssl-devel libyaml-devel libffi-devel readline-devel zlib-devel ncurses-devel sqlite3-devel gcc make wget tar gcc-c++ rpm-build unzip subversion git patch
    ```
  You may already have these packages installed - just install any missing.  
  _**Note:** The `tcl-devel`, `tk-devel` and `gdbm-devel` packages must **NOT** be installed (otherwise the Ruby build process will incorporate them and Omnibus will fail with a Healthcheck dependency error later)_

2. Build Ruby 2+ directly

  _**WARNING: ** Do not perform the Ruby install dependencies step as that specifies the `tcl` / `tk` / `gdbm` libraries. The dependencies above cover the Ruby build process. If you already have `tcl-devel`, `tk-devel` or `gdbm-devel` installed you'll need to remove them for the duration of this build_  
  
  The Ruby version available on the RHEL 6.6 and SLES 11 package repositories is too low level and the version available on RHEL 7 and SLES 12 contains the tcl/tk libraries so build and install Ruby yourself following the instructions [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby).  
  
  Once you have completed the Ruby install continue to follow the process below  

3. Move to the location you wish to store the temporary Omnibus source in and install the basic gems

  ```shell
  cd /<omnibus_source_root>/
  gem install bundler
  gem install rake-compiler
  ```
4. Clone the github Omnibus repositories

  ```shell
  git clone https://github.com/chef/omnibus-chef
  git clone https://github.com/chef/omnibus-software
  ```
  SLES 11/12 Only
  ```shell
  git clone https://github.com/chef/omnibus
  ```
5. **SLES 11/12 Only** Modify the platform file to support RPMs for suse and make bundler use the extracted git repository

    ```shell
    vi omnibus/lib/omnibus/packager.rb
    ```
    Add suse to the packaging list so that 
    ```ruby
    PLATFORM_PACKAGER_MAP = {
      'debian'   => DEB,
      'fedora'   => RPM,
      'rhel'     => RPM,
      'aix'      => BFF,
      'solaris2' => Solaris,
      'windows'  => MSI,
      'mac_os_x' => PKG,
    }.freeze
    ```
    becomes
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
    Then edit the bundler Gemfile to use this local repository
    ```shell
    vi omnibus-chef/Gemfile
    ```
    And change this line from
    ```ruby
    gem 'omnibus', github: 'opscode/omnibus'
    ```
    to 
    ```ruby
    gem 'omnibus', path: '/<omnibus_source_root>/omnibus'
    ```
    _Where `/<omnibus_source_root>/` is the root directory you git cloned into above_
5. Move into the source tree and install additional necessary gems

  ```shell
  cd omnibus-chef
  bundle install --binstubs
  ```
  _**Note:** Bundler will likely complain about being run as root - as we are building an RPM this won't impact the end result_
  
6. Modify the omnibus-chef configuration file

    ```shell
    vi omnibus.rb
    ```
    Turn off S3 caching (to enable a full true build), change the below section:
    ```ruby
    use_s3_caching true
    s3_access_key  ENV['AWS_ACCESS_KEY_ID']
    s3_secret_key  ENV['AWS_SECRET_ACCESS_KEY']
    s3_bucket      'opscode-omnibus-cache'
    ```
    to
    ```ruby
    use_s3_caching false
    #s3_access_key  ENV['AWS_ACCESS_KEY_ID']
    #s3_secret_key  ENV['AWS_SECRET_ACCESS_KEY']
    #s3_bucket      'opscode-omnibus-cache'
    ```
    Turn off git caching (to prevent any commit time errors - only necessary if git isn't configured) by uncommenting the line below
    ```ruby
    # use_git_caching false
    ```
    to
    ```ruby
    use_git_caching false
    ```
    _**Note:** Another solution would be `git config --global user.email "you@example.com"` which will specify an email address which is the minimum git configuration to be able to commit._

8. **SLES Only** you need to copy and edit the libffi support file

  ```shell
  cp ../omnibus-software/config/software/libffi.rb config/software/.
  vi config/software/libffi.rb
  ```
  Then comment out the RHEL && 64 bit check so turning 
  ```ruby
  # On 64-bit centos, libffi libraries are places under /embedded/lib64
  # move them over to lib
  if rhel? && _64_bit?
    # Can't use 'move' here since that uses FileUtils.mv, which on < Ruby 2.2.0-dev
    # returns ENOENT on moving symlinks with broken (in this case, already moved) targets.
    # http://comments.gmane.org/gmane.comp.lang.ruby.cvs/49907
    copy "#{install_dir}/embedded/lib64/*", "#{install_dir}/embedded/lib/"
    delete "#{install_dir}/embedded/lib64"
  end
  ```
  into 
  ```ruby
  # On 64-bit centos, libffi libraries are places under /embedded/lib64
  # move them over to lib
  #if rhel? && _64_bit?
    # Can't use 'move' here since that uses FileUtils.mv, which on < Ruby 2.2.0-dev
    # returns ENOENT on moving symlinks with broken (in this case, already moved) targets.
    # http://comments.gmane.org/gmane.comp.lang.ruby.cvs/49907
    copy "#{install_dir}/embedded/lib64/*", "#{install_dir}/embedded/lib/"
    delete "#{install_dir}/embedded/lib64"
  #end
    ```
    _**Note:** This is unnecessary on 31 bit systems_
7. Copy two files from software to chef

  ```shell
  cp ../omnibus-software/config/software/ruby.rb config/software/.
  cp ../omnibus-software/config/software/cacerts.rb config/software/.
  ```
8.  Modify the ruby.rb file configure command to prevent the server crashing later

  ```shell
  vi config/software/ruby.rb
  ```
  Then change the end of this section 
  ```ruby
  configure_command = ["./configure",
                         "--prefix=#{install_dir}/embedded",
                         "--with-out-ext=dbm",
                         "--enable-shared",
                         "--enable-libedit",
                         "--with-ext=psych",
                         "--disable-install-doc",
                         "--without-gmp",
                         "--disable-dtrace"]
  ```
  to add one last item
  ```ruby
  configure_command = ["./configure",
                         "--prefix=#{install_dir}/embedded",
                         "--with-out-ext=dbm",
                         "--enable-shared",
                         "--enable-libedit",
                         "--with-ext=psych",
                         "--disable-install-doc",
                         "--without-gmp",
                         "--disable-dtrace",
                         "--with-setjmp-type=_setjmp"]

  ```
9. Modify the cacerts.rb file to patch the 3rd party certificate file changing hash

  ```shell
  vi config/software/cacerts.rb
  ```
  Then modify this entry
  ```ruby
  version "2014.08.20" do
     source md5: "c9f4f7f4d6a5ef6633e893577a09865e"
  end
  ```
  to be 
  ```ruby
  version "2014.08.20" do
     source md5: "380df856e8f789c1af97d0da9a243769"
  end

  ```
  _**Note:** This is only necessary because the Omnibus build process and the downloaded certificate does not match correctly, this may be corrected in later versions of the Omnibus build code_
10. Add a post install patch to ensure the ohai package can detect the platform correctly
    
    ```shell
  vi package-scripts/chef/postinst
  ```
  Add the following lines just before the "Thank you for installing Chef!" message in the postinst file

    RHEL 6.6 & 7.1
    ```patch
    patch -p0 << END_OF_FILE
    --- /opt/chef/embedded/lib/ruby/gems/2.1.0/gems/ohai-8.4.0/lib/ohai/plugins/linux/platform.rb   2015-05-26 09:52:54.000000000 -0400
    +++ /opt/chef/embedded/lib/ruby/gems/2.1.0/gems/ohai-8.4.0/lib/ohai/plugins/linux/platform.rb  2015-05-26 11:17:43.510207156 -0400
    @@ -115,7 +115,7 @@
           platform_family "debian"
         when /fedora/, /pidora/
           platform_family "fedora"
    -    when /oracle/, /centos/, /redhat/, /scientific/, /enterpriseenterprise/, /amazon/, /xenserver/, /cloudlinux/, /ibm_powerkvm/, /parallels/ # Note that 'enterpriseenterprise' is oracle's LSB "distributor ID"
    +    when /oracle/, /centos/, /redhat/, /scientific/, /enterpriseenterprise/, /amazon/, /xenserver/, /cloudlinux/, /ibm_powerkvm/, /base/, /parallels/ # Note that 'enterpriseenterprise' is oracle's LSB "distributor ID"
           platform_family "rhel"
         when /suse/
           platform_family "suse"
    END_OF_FILE
    ```
    
    SLES 11 & 12
    ```patch
    patch -p0 << END_OF_FILE
    --- /opt/chef/embedded/lib/ruby/gems/2.1.0/gems/ohai-8.4.0/lib/ohai/plugins/linux/platform.rb   2015-05-26 09:52:54.000000000 -0400
    +++ /opt/chef/embedded/lib/ruby/gems/2.1.0/gems/ohai-8.4.0/lib/ohai/plugins/linux/platform.rb  2015-05-26 11:17:43.510207156 -0400
    @@ -115,7 +115,7 @@
           platform_family "debian"
         when /fedora/, /pidora/
           platform_family "fedora"
    -    when /oracle/, /centos/, /redhat/, /scientific/, /enterpriseenterprise/, /amazon/, /xenserver/, /cloudlinux/, /ibm_powerkvm/, /parallels/ # Note that 'enterpriseenterprise' is oracle's LSB "distributor ID"
    +    when /oracle/, /centos/, /redhat/, /scientific/, /enterpriseenterprise/, /amazon/, /xenserver/, /cloudlinux/, /ibm_powerkvm/, /parallels/ # Note that 'enterpriseenterprise' is oracle's LSB "distributor ID"
           platform_family "rhel"
         when /suse/, /base/
           platform_family "suse"
    END_OF_FILE
    ```
    _**Note:** This is simply adding the `/base/` filter to the platform detection in the file `/opt/chef/embedded/lib/ruby/gems/2.1.0/gems/ohai-8.4.0/lib/ohai/plugins/linux/platform.rb` once the rpm is installed. You may require `patch` on your system before installing the RPM._
11. Create a stub fakeroot script (if necessary)
    
    The Omnibus build process uses fakeroot to produce the rpm regardless of the need, and this is part of the dynamic build process so it cannot be locally changed.  
    If fakeroot is already available on your machine (`which fakeroot`) then the build will use that, however if it is not available it is safest to create a quick stub script:

    ```shell
    echo '"$@"' > /usr/bin/fakeroot
    chmod +x /usr/bin/fakeroot
    ```
13. Run the Omnibus build and produce the RPM

  ```shell
  bundle exec omnibus build chef --log-level=debug --override append_timestamp:false
  ```
  _**Note:** Should the build fail you can use `bundle exec omnibus clean chef` or `bundle exec omnibus clean chef --purge` to clean the build tree before attempting again._

14. The Chef client Omnibus RPM is now available at `/<omnibus_source_root>/omnibus-chef/pkg/`
15. Clean up the stub fakeroot script (if created in step 13)

    ```shell
    rm /usr/bin/fakeroot
    ```