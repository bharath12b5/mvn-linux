#Building Chef client V12.1.2 on Linux on Z RHEL6.5  

##Before you begin   
+ If you don't have one, get a GitHub account from [Join Github](https://github.com/join).
+ Register the server you are using with GitHub using these instructions
[GitHub ssh-keys](https://help.github.com/articles/generating-ssh-keys/)
+ Build as root, or be prepared to sudo many of the following commands
+ If you've built before there may be a directory called .chef in your personal root directory.
If so, make sure it's rwx by "other", plus any contents of that directory

##Prereqs
+ *ensure RHEL 6.5 is installed*  
>         cat /etc/redhat-release  

+ *Install or verify gcc is installed:*   
>         rpm -qa | grep gcc  

+ *Install various prerequisites*
>         yum install readline-devel.s390 readline-devel.s390x git subversion fakeroot  

+ *Get yaml*   
*First, download http://pyyaml.org/download/libyaml/yaml-0.1.5.tar.gz then*  
>         tar -xvf yaml-0.1.5.tar.gz  

+ *Build and install yaml:*  
>         cd yaml-0.1.5  
>         ./configure  
>         make  
>         make install     

+ *Get Ruby (Ruby 2.1.4, built from source from  https://www.ruby-lang.org/en/downloads)*  
*Untar/Unzip:*  
>         wget http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.4.tar.gz   
>         tar -xvf  ruby-2.1.4.tar.gz  

+ *Build and Install Ruby:*  
>         cd  ruby-2.1.4  
>         ./configure  
>         make  
>         make install  

+ *Verify Ruby is installed*    
>         ruby -v  

+ *Get Ruby Gems (gem 2.2.2 from source http://production.cf.rubygems.org/rubygems/rubygems-2.2.2.tgz)*    
*Untar/Unzip:*  
>         tar -xvf  rubygems-2.2.2.tgz     

+ *Build and install Ruby Gems*  
>         cd  rubygems-2.2.2    
>         ruby setup.rb    

+ *Verify Ruby Gems installed*  
>         gem env  
>         gem install bundler  

+ *Check that unzip and svn are installed*  
>         using rpm -qa | grep "unzip\|svn"  

+ *install rake*  
>         gem install rake-compiler  


---------------------------------
##Building chef client:

*Use git repository omnibus-chef for client*
*Note that omnibus-software (subdirectory config/software) is the repository for ruby files to update as noted below*  
  
>         git clone https://github.com/chef/omnibus-chef  
  
*for a specific version, like 12.1.2 you can also download and unzip*  
*https://github.com/chef/omnibus-chef/archive/chef-12.1.2.tar.gz*   
  
>         git clone  https://github.com/chef/omnibus-software      
  
*(NOTE: omnibus-software has no releases)*  
  
>         cd omnibus-chef
>         bundle install --binstubs  
  
###You will need to change:
1. one file in the base directory (omnibus.rb)  
1. two files in config/software (see below)  
1. one file in  package-scripts/chef-server (postinst)  

####1. One file in the base directory needs to be changed: omnibus.rb  
*turn off S3 caching (which will enable a full true build) by commenting out these lines in omnibus.rb*  
>     #use_s3_caching true  
>     #s3_access_key  ENV['AWS_ACCESS_KEY_ID']  
>     #s3_secret_key  ENV['AWS_SECRET_ACCESS_KEY']  
>     #s3_bucket      'opscode-omnibus-cache'  

*add*  
>     use_s3_caching false   

####2. Two files in config/software 
*The following need to be copied from*  
omnibus-software/config/software   
*to*   
omnibus-chef/config/software    
*and modified as noted below*   

+ ruby.rb
+ cacerts.rb

#####ruby.rb   
*need to add  "--with-setjmp-type=_setjmp" to config or server will crash*   
>                        "--disable-dtrace",   
>                        "--with-setjmp-type=_setjmp"]   

*instead of*   
>                        "--disable-dtrace"]   
   
#####cacerts.rb
*Need to use a more recent version, old one is no longer available*    
>     default_version "2015.02.25"

*instead of*   
>     default_version "2014.09.03"

*plus add*    
>     version "2015.02.25" do
>       source md5: "19e7f27540ee694308729fd677163649"
>     end



####3. One file in  package-scripts/chef needs to be changed: postinst   
*The file postinst needs to be changed to include some postinstall modifications to a file in the ohai package (platform.rb)*   
*The purpose of these modifications is to enable ohai to properly sense and report on the Z/390 platform*   
*Just add the following lines just before the "Thank you for installing Chef!" message in the postinst file*   

>        patch -p0 << END_OF_FILE
>        --- /opt/chef/embedded/lib/ruby/gems/2.1.0/gems/ohai-8.1.1/lib/ohai/plugins/linux/platform.rb   2015-03-19 12:30:03.000000000 +0100
>        +++ /opt/chef/embedded/lib/ruby/gems/2.1.0/gems/ohai-8.1.1/lib/ohai/plugins/linux/platform.rb   2015-03-19 12:34:16.000000000 +0100
>        @@ -115,7 +115,7 @@
>               platform_family "debian"
>             when /fedora/, /pidora/
>               platform_family "fedora"
>        -    when /oracle/, /centos/, /redhat/, /scientific/, /enterpriseenterprise/, /amazon/, /xenserver/, /cloudlinux/, /ibm_powerkvm/, /parallels/ # Note that 'enterpriseenterprise' is oracle's LSB "distributor ID"
>        +    when /oracle/, /centos/, /redhat/, /scientific/, /enterpriseenterprise/, /amazon/, /xenserver/, /cloudlinux/, /ibm_powerkvm/, /base/, /parallels/ # Note that 'enterpriseenterprise' is oracle's LSB "distributor ID"
>               platform_family "rhel"
>             when /suse/
>               platform_family "suse"
>        END_OF_FILE

###Kick off the build

>     bundle exec omnibus build chef --log-level=debug --override append_timestamp:false

*This gives copious debugging information plus produces an rpm that doesn't have a timestamp*    
*Cleaning up between builds is done with*    
>     bundle exec omnibus clean chef

*or*    
>     bundle exec omnibus clean chef --purge

*for a total cleansing, including all the gzip files that had been downloaded*    



 



