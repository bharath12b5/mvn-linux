<!---PACKAGE:Rails--->
<!---DISTRO:RHEL 6.x:5.x--->
<!---DISTRO:RHEL 7.x:5.x--->
<!---DISTRO:SLES 11.x:5.x--->
<!---DISTRO:SLES 12.x:5.x--->
<!---DISTRO:Ubuntu 16.x:Distro, 5.x--->

# Building Ruby on Rails

Below versions of Ruby on Rails are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `4.2.6`

The instructions provided below specify the steps to build Rails version 5.0.0 on Linux on the IBM z Systems for RHEL 6.7/7.1/7.2, SLES 11-SP3/12 , SLES12-SP1 and Ubuntu 16.04.  

_**General Notes:**_   
_i) When following the steps below please use a standard permission user unless otherwise specified._     
_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

1. First build Ruby 1.9.3+ (the available yum / zypper version of ruby is too low level) from [these instructions](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby) (For RHEL6.7/7.1/7.2 , SLES11-SP3, SLES12 and SLES12-SP1)

2. Correct the gem environment for a standard user (For RHEL6.7/7.1 , SLES11/12 and SLES12-SP1)

        export GEM_HOME=/home/<USER>/.gem/ruby
        export PATH=/home/<USER>/.gem/ruby/bin:$PATH
    
 Where `<USER>` is the standard user you are installing under

3. Add build dependencies
    
    For RHEL6.7/7.1/7.2:

        sudo yum install -y patch make gcc
    
    For SLES11-SP3, SLES12 and SLES12-SP1:

        sudo zypper install -y patch make gcc
        
    For Ubuntu 16.04:
 
        sudo apt-get install -y ruby ruby-dev patch make gcc zlib1g-dev
 

4. Install Ruby on Rails via gem

      RHEL6.7/7.1/7.2 , SLES11-SP-3, SLES12 and SLES12-SP1
      
          gem install rails -v 5.0
          
      Ubuntu 16.04
           
          sudo gem install rails -v 5.0
    
5. Ruby on Rails is now installed. Verify version with command `rails -v` (Output should be as follows):
     ```
    5.0.0
     ```
	 
