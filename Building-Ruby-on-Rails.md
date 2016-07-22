<!---PACKAGE:Rails--->
<!---DISTRO:RHEL 6.6:5.0.0--->
<!---DISTRO:RHEL 7.1:5.0.0--->
<!---DISTRO:SLES 11:5.0.0--->
<!---DISTRO:SLES 12:5.0.0--->
<!---DISTRO:Ubuntu 16.x:5.0.0--->

# Building Ruby on Rails

Below versions of Ruby on Rails are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `4.2.6`

The instructions provided below specify the steps to build Rails version 5.0.0 on Linux on the IBM z Systems for RHEL 6.6/7.1, SLES 11/12 and Ubuntu 16.04.  

_**General Notes:**_   
_i) When following the steps below please use a standard permission user unless otherwise specified._     
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

1. First build Ruby 1.9.3+ (the available yum / zypper version of ruby is too low level) from [these instructions](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby) (For RHEL6.6/7.1 and SLES11/12)

2. Correct the gem environment for a standard user (For RHEL6.6/7.1 and SLES11/12)

        export GEM_HOME=/home/<USER>/.gem/ruby
        export PATH=/home/<USER>/.gem/ruby/bin:$PATH
    
 Where `<USER>` is the standard user you are installing under

3. Add build dependencies
    
    For RHEL6.6/7.1:

        sudo yum install -y patch make gcc
    
    For SLES11/12:

        sudo zypper install -y patch make gcc
        
    For Ubuntu 16.04:
 
        sudo apt-get install -y ruby ruby-dev patch make gcc zlib1g-dev
 

4. Install Ruby on Rails via gem

      RHEL6.6/7.1 and SLES11/12
      
          gem install rails -v 5.0
          
      Ubuntu 16.04
           
          sudo gem install rails -v 5.0
    
5. Ruby on Rails is now installed. Verify version with command `rails -v` (Output should be as follows):
     ```
    5.0.0
     ```
	 
