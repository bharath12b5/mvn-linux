<!---PACKAGE:Rails--->
<!---DISTRO:RHEL 6.6:4.2.6--->
<!---DISTRO:RHEL 7.1:4.2.6--->
<!---DISTRO:SLES 11:4.2.6--->
<!---DISTRO:SLES 12:4.2.6--->

### Building Ruby on Rails on RHEL6/7 and SLES11/12

# Installing [Ruby on Rails](http://rubyonrails.org/) on RHEL6/7 and SLES11/12

The Ruby on Rails code version 4.2.6 can be installed on a Linux on z System running RHEL6/7 and SLES11/12 by following these instructions:

**NOTE**: When following the steps below please use a standard
permission user unless otherwise specified.

1. First build Ruby 1.9.3+ (the available yum / zypper version of ruby is too low level) from [these instructions](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby)

2. Correct the gem environment for a standard user

        export GEM_HOME=/home/<USER>/.gem/ruby
        export PATH=/home/<USER>/.gem/ruby/bin:$PATH
    
 Where `<USER>` is the standard user you are installing under

3. Add a few additional build dependencies
    
 For RHEL6/7,

        sudo yum install -y patch make gcc
    
 For SLES11/12,

        sudo zypper install -y patch make gcc

4. Install Ruby on Rails via gem

        gem install rails -v 4.2.6
    
5. Ruby on Rails is now installed. Verify version with command `rails -v` (Output should be as follows):
     ```
    4.2.6
     ```