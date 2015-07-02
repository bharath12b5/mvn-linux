# Building Chef Client on RHEL 7.1/6.6 or SLES 12/11

The Chef Client code can be built for a Linux on z System running RHEL 7.1/6.6 or SLES 12/11 by following these instructions (Chef is available at https://www.chef.io/ and the github repository for the client can be found at https://github.com/chef/chef):

_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified._

1. Install the build time dependencies

  For **RHEL 7.1** use the following
  ```shell
  sudo yum install git ruby ruby-devel rubygems rubygem-bundler gcc make
  ```
  For **RHEL 6.6** use the below (note that ruby is not included - that is resolved later)
  ```shell
  sudo yum install git gcc make
  ```
  For **SLES 12** (note the package name differences)
  ```shell
  sudo zypper install git ruby2.1 ruby2.1-devel ruby2.1-rubygem-bundler gcc make
  ```
  And finally for **SLES 11** use the following (again notice the lack of ruby)
  ```shell
  sudo zypper install git gcc make
  ```
  You may already have these packages installed - just install any missing.

2. For RHEL 6.6 and SLES 11 you will need to build Ruby 2+ directly

  The Ruby version available on the RHEL 6.6 and SLES 11 package repositories is too low level, it needs to be at version 1.9.3+, so build and install Ruby yourself following the instructions [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby).
  Once you have completed the Ruby install continue to follow the process below

3. Move to the location you wish to store the Chef source in

  ```shell
  cd /<chef_source_root>/
  ```
3. Clone the github Chef client repository

  ```shell
  git clone https://github.com/chef/chef.git
  ```
4. Move into the source tree

  ```shell
  cd chef
  ```
5. Skip this step if you are on RHEL 7.1, on all other OS correct the gem environment for a standard user

  ```shell
  export GEM_HOME=/home/<USER>/.gem/ruby
  export PATH=/home/<USER>/.gem/ruby/bin:$PATH
  ```
  *Where `<USER>` is the standard user you are installing under.*  
  _**Note:** Run `gem env` to verify the state of the environment, if later on you have issues installing / running ruby gems please ensure the environment is set correctly._

5. Install the required version of the bundler ruby gem

  ```shell
  gem install bundler -v '1.7.3'
  ```
5. Use bundler to install Chef Client's ruby gem dependencies

  ```shell
  bundle install
  ```
6. Build the Chef Client ruby gem packages

  ```shell
  rake gem
  ```
7. Install the gem you just built

  ```shell
  ls pkg/*.gem | grep -v mingw32 | xargs gem install
  ```
  Alternatively `ls pkg` and `gem install pkg/chef-<VERSION>.gem`  
  _**Note:** The mingw32 gem is for windows and can be safely ignored._

8. Chef client is now built and installed (verify with `chef-client` or `knife`)

#Testing Chef Client
If you'd like to test the Chef client you've just built and installed, just follow the steps below:

1. Run the test suite

  ```shell
  cd /<chef_source_root>/chef/
  bundle exec rake spec
  ```
  _**Note:** This may fail as there can be a dependency on rdoc/task in the Rakefile, if that happens just comment out the rdoc/task line as below_
  ```rake
  require 'rubygems/package_task'
  require 'rdoc/task'
  require_relative 'tasks/rspec'
  ```
  can be changed to be 
  ```rake
  require 'rubygems/package_task'
  #require 'rdoc/task'
  require_relative 'tasks/rspec'
  ```
2. Visit https://github.com/chef/chef#testing for more