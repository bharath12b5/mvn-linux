Ruby is available on RHEL 7.1/6.6 or SLES 12/11 but only in older versions, should you require a different version from those listed below you will need to build it.  
* RHEL 7.1 has `2.0.0p598`
* RHEL 6.6 has `1.8.7p374`
* SLES 12 has `2.1.2p95`
* SLES 11.3 has `1.8.7p357`

Packages that use ruby may require a specific version, in particular, having [RubyGems](http://en.wikipedia.org/wikiRubyGems) is essential for large packages using Ruby (Ruby version >= 1.9)
The authoritative source for most things "ruby" is [here](http://www.ruby-lang.org) for the latest "stable" Ruby version see [download link](http://www.ruby-lang.org/en/downloads)

_**NOTE:** When following the steps below please use a standard permission user unless otherwise specified._

1. Install the dependencies for your specific platform

    RHEL 7.1 & 6.6
    ```shell
    sudo yum install bison flex openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite-devel gcc make wget tar
    ```
    SLES 12
    ```shell
    sudo zypper install bison flex libopenssl-devel libyaml-devel libffi48-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel gcc make wget tar
    ```
    SLES 11
    ```shell
    sudo zypper install bison flex libopenssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel tcl-devel tk-devel sqlite3-devel gcc make wget tar
    ```

2. Download and unpack the Ruby 2.2.3 source code

    ```shell
    cd /<source_root>/
    wget http://cache.ruby-lang.org/pub/ruby/ruby-2.2.3.tar.gz
    tar zxf ruby-2.2.3.tar.gz
    cd ruby-2.2.3
    ```
3. Configure and build Ruby

    ```shell
    ./configure
    make
    ```

4. Test the results (_optional_) and install ruby

    ```shell
    make test
    ```
    _**Note:** The tests should not report any failures_
    ```shell
    sudo make install
    ```
    _The default location for installation when building from source is in /usr/local --  if ruby was installed not by building from source, it would be in /usr -- there are other ways to have multiple versions of ruby (or other packages installed) but taking the default is the simplest -- when installing things that require the from source version of ruby, make sure the appropriate path (/usr/local/bin) is first in the PATH environment variable_
5. Verify that ruby is available

    ```
    ruby -v
    gem env
    ```
    _**Note:** Rubygems support is contained as part of this build, so `gem env` should work_
