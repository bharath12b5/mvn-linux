RHEL 7 currently carries the 9.2 line of PostgreSQL. The community-contributed [PostgreSQL RPM Repository](http://yum.postgresql.org/) provides the latest 9.4 binaries, but only for x86; publishing the z binaries through the official RPM repository is a work in progress. In the mean time, PostgreSQL 9.4 can be built for Linux on z from source RPMs.

The following instructions have been tested on RHEL 7.1.

1. Download [the latest 9.4 source RPM for RHEL 7](http://yum.postgresql.org/srpms/9.4/redhat/rhel-7-x86_64/). As of this writing, the file to download is postgresql94-9.4.4-1PGDG.rhel7.src.rpm, however the exact version number (e.g. 9.4.4-1) may change with time.

2. Install the dependencies (as root):

        yum install perl-ExtUtils-Embed perl-ExtUtils-MakeMaker \
                    python-devel tcl-devel readline-devel zlib-devel \
                    openssl-devel krb5-devel e2fsprogs-devel libxml2-devel \
                    libxslt-devel pam-devel libuuid-devel openldap-devel \
                    openjade opensp docbook-dtds docbook-style-dsssl

3. Rebuild the binary RPM from the source RPM:

        rpmbuild --rebuild postgresql94-9.4.*PGDG.rhel7.src.rpm

   **(Optional)** For improved performance, rebuild with additional compiler flags:

        rpmbuild --rebuild --define 'optflags -O3 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -funroll-loops -m64 -march=z196 -mtune=zEC12' postgresql94-9.4.*PGDG.rhel7.src.rpm

4. The binary RPMs will be created in _$HOME_/rpmbuild/RPMS/s390x/. They can be simply installed with the `rpm` command (as root):

        cd $HOME/rpmbuild/RPMS/s390x/
        rpm -i postgresql94-9.4.*PGDG.el7.s390x.rpm \
               postgresql94-server-9.4.*PGDG.el7.s390x.rpm \
               postgresql94-libs-9.4.*PGDG.el7.s390x.rpm

   Other RPMs in the same directory are optional and can be installed as well if you need them.

5. The build process may have set the permissions on the directory /usr/pgsql-9.4/lib/ incorrectly. The permissions must be updated (as root) to allow users to read the PostgreSQL libraries:

        chmod o+r /usr/pgsql-9.4/lib/

6. Before starting the database server, you must initialize the cluster. This is done using a command line tool, which is designed for the RPMs. Run this command as root:

        /usr/pgsql-9.4/bin/postgresql94-setup initdb

7. Now, you can start up the server:

        systemctl start postgresql-9.4.service