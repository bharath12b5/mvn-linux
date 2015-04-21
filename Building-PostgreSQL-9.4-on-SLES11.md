SLES 11 SP3 currently carries the 9.1 line of PostgreSQL. The official [PostgreSQL RPM Repository](http://yum.postgresql.org/) provides the latest 9.4 binaries, but only for x86; publishing the z binaries through the official RPM repository is a work in progress. In the mean time, PostgreSQL 9.4 can be built for Linux on z from source RPMs.

The following instructions have been tested on SLES 11 SP3.

1. Building PostgreSQL requires the uuid-devel package, which is not included in the SLES 11 SP3 package repository. However, the source RPM uuid-1.6.2-4.2.3.1.src.rpm can be found on disc 2 of the installation media (under suse/src/). Rebuild this source RPM (as root):

        rpmbuild --rebuild uuid-1.6.2-4.2.3.1.src.rpm

2. Three binary RPMs will be created in /usr/src/packages/RPMS/s390x/. Install two of them (as root):

        cd /usr/src/packages/RPMS/s390x
        rpm -i uuid-devel-1.6.2-4.2.3.1.s390x.rpm libossp-uuid16-1.6.2-4.2.3.1.s390x.rpm

3. Download the following three PostgreSQL RPMs from openSUSE:

    - [postgresql-init](http://download.opensuse.org/repositories/server:/database:/postgresql/SLE_11_SP3/noarch/postgresql-init-9.4-46.2.noarch.rpm)
    - [postgresql94-libs](http://download.opensuse.org/repositories/server:/database:/postgresql/SLE_11_SP3/src/postgresql94-libs-9.4.1-7.2.src.rpm)
    - [postgresql94](http://download.opensuse.org/repositories/server:/database:/postgresql/SLE_11_SP3/src/postgresql94-9.4.1-7.2.src.rpm)

4. Install the dependencies (as root):

        zypper install gettext-devel ncurses-devel pam-devel pwdutils \
                       python-devel readline-devel tcl-devel timezone \
                       zlib-devel systemd fdupes krb5-devel libxslt-devel \
                       openldap2-devel openssl-devel pkg-config update-alternatives

5. Install the postgresql-init package downloaded above (as root):

        rpm -i postgresql-init-9.4-46.2.noarch.rpm

6. Rebuild the binary RPMs from the two downloaded source RPMs (as root):

        rpmbuild --rebuild postgresql94-9.4.1-7.2.src.rpm
        rpmbuild --rebuild postgresql94-libs-9.4.1-7.2.src.rpm

7. Install the generated binary RPMs (as root):

        cd /usr/src/packages/RPMS/s390x
        rpm -i postgresql94-9.4.1-7.2.s390x.rpm \
               postgresql94-contrib-9.4.1-7.2.s390x.rpm \
               postgresql94-server-9.4.1-7.2.s390x.rpm \
               libpq5-9.4.1-7.2.s390x.rpm

   Other RPMs in the same directory are optional and can be installed as well if you need them.

8. Make sure the /var/lib/postgresql/ directory exists. Run these commands as root:

        mkdir /var/lib/postgresql
        chown postgres:postgres /var/lib/postgresql

9. Starting the database server for the first time will initialize the cluster automatically. To start the server, run this command as root:

        service postgresql start