# Building Fluentd

The following build instructions have been tested with Fluentd-0.12.8 on RHEL7 and SLES12 on IBM z Systems.

###Version
0.12.8
###Section 1: Install the following Dependencies
*	git (or git-core)
*	ruby
*	gcc
*	gcc-c++
*	kernel-devel
*	ruby-devel
*	openssl
*	make
*	timezone

RHEL7:
```
yum install -y git \
	ruby \
	gcc \
	gcc-c++ \
	kernel-devel \
	ruby-devel \
	openssl \
	make \
	timezone
```

SLES12:
```
zypper install -y git-core \
	make \
	ruby \
	gcc \
	gcc-c++ \
	kernel-devel \
	ruby-devel \
	timezone
```
###Section 2. Build and Install Fluentd
1. Re-install the ca certificates for openssl framework.
        yum reinstall -y ca-certificates
2. Install the bundler using gem
        gem install bundler
    Can specify the version of the bundler (Optional)

    For instance,
        gem install bundler -v="1.9.4"
    Set the Bundler Path as per the installed version
        export PATH=$PATH:/usr/local/share/gems/gems/bundler-1.9.4/bin
		
3. Create a fluentd user
        useradd -d /home/fluentd -m -s /bin/bash fluentd -p fluentd
		
4. Get the fluentd source and clone as fluentd user
        cd /home/fluentd
        git clone https://github.com/fluent/fluentd
        cd /home/fluentd/fluentd
		
5. Install bundle
        bundle install --path=./vendor/bundle
        bundle exec rake
		
6. Install fluentd as root user
        cd /home/fluentd/fluentd
        gem install pkg/fluentd-0.12.8.gem
    Set fluentd environment path
        export PATH=$PATH:/usr/local/share/gems/gems/fluentd-0.12.8/bin
        fluentd --setup ./fluent
		
7. By default, fluentd configuration file is created at ./fluent/fluent.conf

##References:
https://github.com/fluent/fluentd