# Building Doxygen

Recipe for Doxygen version 1.8.9.1.

The following build instructions have been tested with Doxygen 1.8.9.1 on RHEL7 and SLES12 on IBM z Systems.

### Version
1.8.9.1

###Section 1: Install the following Dependencies
*	git (or git-core)
*	flex
*	bison
*	gcc
*	gcc-c++
*	texlive-bibtex-bin
*	make
*	perl-Test-Simple.noarch
*	openssl
*	gmake
*	GraphViz
*	python-xml
*	libxml2-tools

RHEL7:
```
	yum install -y git \
		flex \
		bison \
		gcc \
		gcc-c++ \
		texlive-bibtex-bin \
		make \
		perl-Test-Simple.noarch \
		openssl \
		gmake \
		GraphViz \
		python-xml \
		libxml2-tools
```

SLES12:
```
	zypper install -y git-core \
		gmake \
		GraphViz \
		flex \
		bison \
		gcc \
		gcc-c++ \
		python-xml \
		libxml2-tools \
		texlive-bibtex-bin \
		make \
		perl-Test-Simple.noarch 
```
###Section 2. Build and install Doxygen
1. Re-install the ca certificates for openssl framework.
        yum reinstall -y ca-certificates
2. Get the source
        git clone https://github.com/doxygen/doxygen.git/
3. Configure the build, without options results in installing the doxygen to default location
        cd doxygen
        ./configure --prefix=<build-location>--exec-prefix=<build-location>
4. Build the source
        make
5. Run the functional test suite
        make test
6. Install the binaries
        make install

##References:
https://github.com/doxygen/doxygen		