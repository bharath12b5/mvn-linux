SLES 11 SP3 currently carries the 9.1 line of PostgreSQL. The community-contributed [PostgreSQL RPM Repository](http://yum.postgresql.org/) provides the latest 9.4 binaries, but only for x86; publishing the z binaries through the official RPM repository is a work in progress. In the mean time, PostgreSQL 9.4 can be built for Linux on z from source RPMs.

The following instructions have been tested on SLES 11 SP3.

1. Visit the [PostgreSQL package repository for SLES 11 SP3](http://download.opensuse.org/repositories/server:/database:/postgresql/SLE_11_SP3/) and download the latest versions of the following packages:

    - postgresql-init (under the "noarch" directory)
    - postgresql94-libs (under the "src" directory)
    - postgresql94 (under the "src" directory)

   The actual file names would be something like: postgresql-init-9.4-46.5.noarch.rpm, postgresql94-libs-9.4.4-2.3.src.rpm, and postgresql94-9.4.4-2.3.src.rpm. The version numbers may change with time.

2. Install the dependencies (as root):

        zypper install gettext-devel ncurses-devel pam-devel pwdutils \
                       python-devel readline-devel tcl-devel timezone \
                       libuuid-devel zlib-devel systemd fdupes krb5-devel \
                       libxslt-devel openldap2-devel openssl-devel pkg-config \
                       update-alternatives bison flex

   Be sure to [add the SLE-SDK repositories as installation sources for zypper](https://www.novell.com/support/kb/doc.php?id=7015337); otherwise some of the above packages may appear to be unavailable.

3. Install the postgresql-init package downloaded above (as root):

        rpm -i postgresql-init-9.4-*.noarch.rpm

4. Rebuild the binary RPMs from the two downloaded source RPMs (as root):

        rpmbuild --rebuild postgresql94-9.4.*.src.rpm
        rpmbuild --rebuild postgresql94-libs-9.4.*.src.rpm

5. The binary RPMs will be created in /usr/src/packages/RPMS/s390x/. They can be simply installed with the `rpm` command (as root):

        cd /usr/src/packages/RPMS/s390x
        rpm -i postgresql94-9.4.*.s390x.rpm \
               postgresql94-contrib-9.4.*.s390x.rpm \
               postgresql94-server-9.4.*.s390x.rpm \
               libpq5-9.4.*.s390x.rpm

   Other RPMs in the same directory are optional and can be installed as well if you need them.

6. Make sure that the /var/run/postgresql/ directory exists and has the appropriate permissions, by running these commands as root:

        mkdir -p /var/run/postgresql
        chown postgres:postgres /var/run/postgresql

7. Starting the database server for the first time will initialize the cluster automatically. To start the server, run this command as root:

        service postgresql start