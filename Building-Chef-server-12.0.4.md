#Building Chef server 12.0.4 on RHEL 6.5 on z Systems
##Before you begin   
+ Download the Java SDK from
[IBM developerWorks](https://www.ibm.com/developerworks/java/jdk/linux/download.html) and put it in a local repository. As an example, for this recipe we will use this URL:

   > http://tlbgsa.ibm.com/˜example/public/ibm-java-jre-7.0-7.1-s390x.tgz

   Note that this step cannot be automated because developerWorks requires user information.
+ If you do not have a GitHub account, get one from [Join Github](https://github.com/join).
+ Register the server you are using with GitHub using these instructions: [GitHub ssh-keys](https://help.github.com/articles/generating-ssh-keys/).
+ Build as root, or be prepared to sudo many of the following commands.
+ If you have built Chef before, there may be a directory called .chef in your personal root directory.
If so, make sure that the directory (plus any content underneath it) is world-writable

        chmod -R o+rwx ~/.chef

##Prereqs
+ Ensure RHEL 6.5 is installed:

        cat /etc/redhat-release  

+ Install or verify gcc is installed:

        rpm -qa | grep gcc
        yum install gcc

+ Install other pre-requisites:

        yum install readline-devel readline-devel git svn fakeroot unzip

+ Download and unpack yaml source code:

        wget http://pyyaml.org/download/libyaml/yaml-0.1.5.tar.gz
        tar -xvf yaml-0.1.5.tar.gz  

+ Build and install yaml:

        cd yaml-0.1.5
        ./configure
        make
        make install

+ Download and unpack Ruby 2.1.4 (built from source from  https://www.ruby-lang.org/en/downloads):

        wget http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.4.tar.gz   
        tar -xvf ruby-2.1.4.tar.gz  

+ Build and install Ruby:

        cd  ruby-2.1.4  
        ./configure  
        make  
        make install  

+ Verify Ruby is installed:

        ruby -v

+ Download and unpack Ruby Gems 2.2.2:

        wget http://production.cf.rubygems.org/rubygems/rubygems-2.2.2.tgz
        tar -xvf  rubygems-2.2.2.tgz

+ Build and install Ruby Gems:

        cd  rubygems-2.2.2    
        ruby setup.rb    

+ Verify Ruby Gems are installed:

        gem env  
        gem install bundler  

+ Install rake:

        gem install rake-compiler  

---------------------------------
##Building Chef server

*Use git repository opscode-omnibus for server, (omnibus-chef is for the client)*
*Note that omnibus-software (subdirectory config/software) is the repository for ruby files to update as noted below*  
  
>         git clone https://github.com/chef/opscode-omnibus  
  
*for a specific version, like 12.0.4 you can also download and unzip*  
*https://github.com/chef/opscode-omnibus/releases/tag/12.0.4*   
  
>         git clone https://github.com/chef/omnibus-software       
  
*(NOTE: omnibus-software has no releases)*  
  
>         cd opscode-omnibus  
>         bundle install --binstubs  
  
###You will need to change:
1. one file in the recipes directory (opscode-solr4.rb)  
1. one file in the base directory (omnibus.rb)  
1. several files in config/software (see below)  
1. one file in  package-scripts/chef-server (postinst)  

####1. One file in the recipes directory needs to be changed: opscode-solr4.rb.  
*The file is*  
    files/private-chef-cookbooks/private-chef/recipes/opscode-solr4.rb  
*to change it execute this*  
>     sed -i 's/Xloggc/Xverbosegclog/g' opscode-omnibus/files/private-chef-cookbooks/private-chef/recipes/opscode-solr4.rb  

*otherwise ibm java doesn't understand Xloggc and won't start*   
####2. One file in the base directory needs to be changed: omnibus.rb  
*turn off S3 caching (which will enable a full true build) by commenting out these lines in omnibus.rb*  
>     #use_s3_caching true  
>     #s3_access_key  ENV['AWS_ACCESS_KEY_ID']  
>     #s3_secret_key  ENV['AWS_SECRET_ACCESS_KEY']  
>     #s3_bucket      'opscode-omnibus-cache'  

*add*  
>     use_s3_caching false   

####3. Several files in config/software 
*Of the files in opscode-omnibus/config/software that need to be changed, the following are already in the that directory (but need modification):*   
+ openresty-lpeg.rb  
+ opscode-solr4.rb  
+ libossp-uuid.rb  

*Whereas the following need to be copied from*  
omnibus-software/config/software to opscode-omnibus/config/software  
*and modified as noted below*  

+ ruby.rb
+ nodejs.rb
+ cacerts.rb
+ sqitch.rb
+ server-jre.rb
+ openresty.rb

#####openresty-lpeg.rb    
*need to change LUADIR location because jit is not installed*   
>     make "LUADIR=#{install_dir}/embedded/lua/include/", env: env

*instead of*   
>     make "LUADIR=#{install_dir}/embedded/luajit/include/luajit-2.0", env: env   

#####opscode-solr4.rb
*change url for lucene solr(old one has gone away). Change*   
>     source url: "http://www.dsgnwrld.com/am/lucene/solr/4.5.1/solr-#{version}.tgz",   

*to*   
>     source url: "https://archive.apache.org/dist/lucene/solr/4.5.1/solr-#{version}.tgz",   

#####libossp-uuid.rb   
*ftp server no longer available. Change*  
>     source url: "ftp://ftp.ossp.org/pkg/lib/uuid/uuid-#{version}.tar.gz"   
   
*to*   
>     source url: "https://gnome-build-stage-1.googlecode.com/files/uuid-1.6.2.tar.gz",   
>            md5: "5db0d43a9022a6ebbbc25337ae28942f"   
   
#####ruby.rb   
*need to add  "--with-setjmp-type=_setjmp" to config or server will crash*   
>                        "--disable-dtrace",   
>                        "--with-setjmp-type=_setjmp"]   

*instead of*   
>                        "--disable-dtrace"]   
   
#####nodejs.rb     
*need to use opensource node.js modified for Z*   
    
        NOTE: in the following, the line   
            env["LINK"]="g++"   
        is optional -- it's only needed if you are building into an nfs mounted filesystem
        (without it in that circumstance, flock throws an error)
    
*Change the code in the body of nodejs.rb to the following*    
>     name "nodejs"   
>     default_version "0.10.36"   
>    
>     dependency "python"   
>    
>     version "0.10.36" do   
>       source md5: "02de00cb56c976f71a5a9eb693511fe7"   
>     end   
>    
>   
>     source url: "https://github.com/andrewlow/node/archive/V8-3.14.5.9-Node.js-#{version}-201501281023.tar.gz"   
>   
>     relative_path "node-8-3.14.5.9-Node.js-#{version}-201501281023"   
>   
>     build do   
>       env = with_standard_compiler_flags(with_embedded_path)   
>       env["LINK"]="g++"   
>       command "#{install_dir}/embedded/bin/python ./configure" \   
>               " --dest-cpu=s390x --prefix=#{install_dir}/embedded", env: env   
>    
>       make "-j #{workers}", env: env   
>       make "install", env: env   
>     end   

#####cacerts.rb
*Need to use a more recent version, old one is no longer available*    
>     default_version "2015.02.25"

*instead of*   
>     default_version "2014.09.03"

*plus add*    
>     version "2015.02.25" do
>       source md5: "19e7f27540ee694308729fd677163649"
>     end

#####sqitch.rb    
*the old version is no longer available, so use*   
>     default_version "0.999"
>            md5: "b3a9cac1254e0e90e4cc09fc84a66c93"

*instead of*  
>     default_version "0.973"
>            md5: "0994e9f906a7a4a2e97049c8dbaef584"

#####server-jre.rb   
*need to use the IBM jre for Z. There is no "server-jre" available on the ibm site, so our server-jre
is just the IBM jre. server-jre.rb copies the jre into the right directories:*    
>     name "server-jre"
>     
>     dependency "rsync"
>     
>     jre_dir = "#{install_dir}/embedded/jre"
>     
>     build do
>       command "wget http://tlbgsa.ibm.com/˜example/public/ibm-java-jre-7.0-7.1-s390x.tgz"
>       command "mkdir -p #{jre_dir}"
>       command "tar xzf ibm-java-jre-7.0-7.1-s390x.tgz"
>       command "#{install_dir}/embedded/bin/rsync -a ./jre #{install_dir}/embedded"
>       command "chmod +x #{install_dir}/embedded/jre/bin/*"
>     end

#####openresty.rb    
*in the sequence of build parameters, take out all the jits and asms (not present on Z)*    
>     '--with-md5-asm',<--- remove
>     '--with-sha1-asm',<--- remove
>     '--with-pcre-jit',<--- remove
>     '--with-luajit', <--- remove

####4. One file in  package-scripts/chef-server needs to be changed: postinst   
*The file postinst needs to be changed to include some postinstall modifications of a couple files in the ohai package (cpu.rb and platform.rb)*   
*The purpose of these modifications is to enable ohai to properly sense and report on the Z/390 platform*   
*Just add the following lines just before the block of "next-step/welcome" messages in the postinst file*   

>        patch -p0 << END_OF_FILE
>        --- /opt/opscode/embedded/service/gem/ruby/2.1.0/gems/ohai-6.22.0/lib/ohai/plugins/linux/cpu.rb 2015-03-18 16:03:46.857938737 +0100
>        +++ /opt/opscode/embedded/service/gem/ruby/2.1.0/gems/ohai-6.22.0/lib/ohai/plugins/linux/cpu.rb 2015-03-18 16:09:58.097942687 +0100
>        @@ -30,7 +30,12 @@
>             current_cpu = \$1
>             cpu_number += 1
>           when /vendor_id\s+:\s(.+)/
>        -    cpuinfo[current_cpu]["vendor_id"] = \$1
>        +    vendor_id = \$1
>        +    if vendor_id =˜ (/IBM\/S390/)
>        +      cpuinfo["vendor_id"] = vendor_id
>        +    else
>        +      cpuinfo[current_cpu]["vendor_id"] = vendor_id
>        +    end
>           when /cpu family\s+:\s(.+)/
>             cpuinfo[current_cpu]["family"] = \$1
>           when /model\s+:\s(.+)/
>        @@ -52,6 +57,21 @@
>             cpuinfo[current_cpu]["cache_size"] = \$1
>           when /flags\s+:\s(.+)/
>             cpuinfo[current_cpu]["flags"] = \$1.split(' ')
>        +  when /bogomips per cpu:\s(.+)/
>        +    cpuinfo["bogomips per cpu"] = \$1
>        +  when /features\s+:\s(.+)/
>        +    cpuinfo["features"] = \$1.split(' ')
>        +  when /processor\s(\d):\s(.+)/
>        +    current_cpu = \$1
>        +    cpu_number += 1
>        +    cpuinfo[current_cpu] = Mash.new
>        +    current_cpu_info = \$2.split(',')
>        +    for i in current_cpu_info
>        +      name_value = i.split('=')
>        +      name = name_value[0].strip
>        +      value = name_value[1].strip
>        +      cpuinfo[current_cpu][name] = value
>        +    end
>           end
>        end
>        END_OF_FILE
>        
>        patch -p0 << END_OF_FILE
>        --- /opt/opscode/embedded/service/gem/ruby/2.1.0/gems/ohai-6.22.0/lib/ohai/plugins/linux/platform.rb    2015-03-18 11:04:30.217941332 +0100
>        +++ /opt/opscode/embedded/service/gem/ruby/2.1.0/gems/ohai-6.22.0/lib/ohai/plugins/linux/platform.rb    2015-03-18 11:05:31.607942099 +0100
>        @@ -100,7 +100,7 @@
>             platform_family "debian"
>           when /fedora/
>             platform_family "fedora"
>        -  when /oracle/, /centos/, /redhat/, /scientific/, /enterpriseenterprise/, /amazon/, /xenserver/ # Note that 'enterpriseenterprise' is oracle's LSB "distributor ID"
>        +  when /oracle/, /centos/, /redhat/, /scientific/, /enterpriseenterprise/, /amazon/, /xenserver/, /base/ # Note that 'enterpriseenterprise' is oracle's LSB "distributor ID"
>             platform_family "rhel"
>           when /suse/
>             platform_family "suse"
>        END_OF_FILE

###Kick off the build

>     bin/omnibus build chef-server --log-level=debug --override append_timestamp:false    

*This gives copious debugging information plus produces an rpm that doesn't have a timestamp*    
*Cleaning up between builds is done with*    
>     bin/omnibus clean chef-server    

*or*    
>     bin/omnibus clean chef-server --purge    

*for a total cleansing, including all the gzip files that had been downloaded*    

*NOTE it is equivalent to use this command to build*    
bundle exec omnibus build chef-server --log-level=debug --override append_timestamp:false    

##The Whitelist

*After the build finishes building its components, it does a health check. It may fail the health check because of missing items on the "whitelist"*    

*You will now find a directory named something like....*    

/usr/local/lib/ruby/gems/2.1.0/bundler/gems/omnibus-51ca37c7b299/lib/omnibus/    

*containing a file health_check.rb*    
*NOTE: the directory may have different alphanumerics in its name than 51ca37c7b299*    

*In health_check.rb you need to add the following list of files to the whitelist (after the items already there)*    

>     WHITELIST_LIBS = [
>      . . .
>           /libmawt\.so/,
>           /libjvm\.so/,
>           /libj9thr26\.so/,
>           /libj9hookable26\.so/,
>           /libXext\.so/,
>           /libX11\.so/,
>           /libXrender\.so/,
>           /libXft\.so/,
>           /libXtst\.so/,
>           /libXi\.so/,
>           /libfreetype\.so/,
>           /libXau\.so/,
>           /libexpat\.so/,
>           /libfontconfig\.so/,
>           /libxcb\.so/,
>           /libreadline\.so/,
>           /libhistory\.so/
>         ].freeze

*After building, install the rpm to test it.*    
*The server is controlled with the command chef-server-ctl*    
*to start:*    
>         chef-server-ctl start    

*to test*    
>         chef-server-ctl test    
