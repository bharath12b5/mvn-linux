This recipe will cover recent Ruby release built on both SLES12 and RHEL 7.
Ruby 2.1.2p95 is available for SLES 12. Ruby 2.0.0p598 is available for RHEL 7.
if Ruby 2.1.2p95 or Ruby 2.0.0p598  is good enough, just do "zypper install ruby" on SLES or replace zypper with yum on RHEL. In this document, zypper will be used as an example, use yum on RHEL instead. Packages that use ruby may require a specific version, in particular, having [RubyGems](http://en.wikipedia.org/wikiRubyGems) is essential for large packages using Ruby (Ruby version >= 1.9)
The authoritative source for most things "ruby" is [here](http://www.ruby-lang.org) for the latest "stable" Ruby version see [download link](http://www.ruby-lang.org/en/downloads)

This recipe is for the current (2015-03-14) "stable" version (2.2.1p85).
NOTE: One needs to be root to install packages in system directories.
For full function, Ruby has some pre-requisites that should be installed before building from source.

bison and flex (tools for building parsers/compilers)

        zypper install bison
        zypper install flex

OpenSSL - an open source implementation of SSL and TLS

        zypper install openssl-devel

YAML - human readable data serialization

        zypper install libyaml-devel

libffi - interfaces for calling natively compiled functions

        zypper install libffi-devel

GNU readline (package for allowing input line editing)

        zypper install readline-devel

ZLIB - data compression library

        zypper install zlib-devel

GNU database system

        zypper install gdbm-devel

NCURSES - library for text-based user interfaces

        zypper install ncurses-devel

Tcl and Tk (a popular scripting language and graphical interface to same)

        zypper install tcl-devel
        zypper install tk-devel

sqlite standalone database library

        zypper install sqlite-devel

Depending on what happens to be installed on your system, there may be others, see below

Obtain the Ruby source code from

        wget http://cache.ruby-lang.org/pub/ruby/ruby-2.2.1.tar.gz

unzip the tarball and change directory

        tar zxf ruby-2.2.1.tar.gz
        cd ruby-2.2.1

run configure

        ./configure

run make

        make

run tests - should be no failures

        make test

The default location for installation when building from source is in /usr/local --  if ruby was installed not by building from source, it would be in /usr -- there are other ways to have multiple versions of ruby (or other packages installed) but taking the default is the simplest -- when installing things that require the from source version of ruby, make sure the appropriate path (/usr/local/bin) is first in the PATH environment variable

        make install