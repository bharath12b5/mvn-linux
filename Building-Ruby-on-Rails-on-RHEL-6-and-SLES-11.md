# Installing [Ruby on Rails](http://rubyonrails.org/) on RHEL 6.6 or SLES 11

The Ruby on Rails code can be installed on a Linux on z System running RHEL 6.6 or SLES 11 by following these instructions:

_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified._

1. First build Ruby 1.9.3+ (the available yum / zypper version of ruby is too low level) from [these instructions](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby)
2. Correct the gem environment for a standard user
  ```shell
  export GEM_HOME=/home/<USER>/.gem/ruby
  export PATH=/home/<USER>/.gem/ruby/bin:$PATH
  ```
    _Where `<USER>` is the standard user you are installing under_

3. Add a few additional build dependencies  
    On RHEL 6.6
  ```shell
  sudo yum install patch make gcc
  ```
    And on SLES 11
  ```shell
  sudo zypper install patch make gcc
  ```
    You may already have some of these installed, simply install any missing packages.

4. Install Ruby on Rails via gem
  ```shell
  gem install rails
  ```
    _**Note:** this will automatically install (and potentially build) a number of dependencies leaving you with rails installed_

5. Ruby on Rails is now installed (verify with `rails -v`)