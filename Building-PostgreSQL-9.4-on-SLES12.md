SLES 12 currently carries the 9.3 line of PostgreSQL. The official [PostgreSQL RPM Repository](http://yum.postgresql.org/) provides the latest 9.4 binaries, but only for x86; publishing the z binaries through the official RPM repository is a work in progress. In the mean time, PostgreSQL 9.4 can be built for Linux on z from source RPMs.

The following instructions have been tested on SLES 12.

1. Download the following 3 RPMs from openSUSE:

    - [postgresql-init](http://download.opensuse.org/repositories/server:/database:/postgresql/SLE_12/noarch/postgresql-init-9.4-46.1.noarch.rpm)
    - [postgresql94-libs](http://download.opensuse.org/repositories/server:/database:/postgresql/SLE_12/src/postgresql94-libs-9.4.1-7.1.src.rpm)
    - [postgresql94](http://download.opensuse.org/repositories/server:/database:/postgresql/SLE_12/src/postgresql94-9.4.1-7.1.src.rpm)

2. Install the dependencies (as root):

        zypper install gettext-devel ncurses-devel pam-devel pwdutils \
                       python-devel readline-devel tcl-devel timezone \
                       uuid-devel zlib-devel systemd fdupes krb5-devel \
                       libxslt-devel openldap2-devel openssl-devel pkg-config \
                       update-alternatives

3. Install the postgresql-init package downloaded above (as root):

        rpm -i postgresql-init-9.4-46.1.noarch.rpm

4. Rebuild the binary RPMs from the two downloaded source RPMs (as root):

        rpmbuild --rebuild postgresql94-9.4.1-7.1.src.rpm
        rpmbuild --rebuild postgresql94-libs-9.4.1-7.1.src.rpm

5. The binary RPMs will be created in /usr/src/packages/RPMS/s390x/. They can be simply installed with the `rpm` command (as root):

        cd /usr/src/packages/RPMS/s390x
        rpm -i postgresql94-9.4.1-7.1.s390x.rpm \
               postgresql94-contrib-9.4.1-7.1.s390x.rpm \
               postgresql94-server-9.4.1-7.1.s390x.rpm \
               libpq5-9.4.1-7.1.s390x.rpm

   Other RPMs in the same directory are optional and can be installed as well if you need them.

6. Starting the database server for the first time will initialize the cluster automatically. To start the server, run this command as root:

        systemctl start postgresql.service