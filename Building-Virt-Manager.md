[KVM for IBM z Systems](http://www-03.ibm.com/systems/z/solutions/virtualization/kvm/) became generally available in September 2015. [virt-manager](https://virt-manager.org/) (which is based on the [libvirt](http://libvirt.org/) API) is a popular GUI application for creating and managing KVM guests. Thanks to recent effort by the virt-manager and libvirt communities, the latest version now has the functionality to create and destroy guests running on KVM on z.

The following instructions show how to build libvirt and virt-manager on an x86-64 system, and how to use it to connect to a KVM hypervisor running on IBM z Systems, and manage its guests.

## Building libvirt

The core functionalities of virt-manager are implemented in libvirt, so it must be built first. The following steps show how to build and install libvirt. These instructions have been tested on RHEL 7.1 on x86.

1. Install libvirt dependencies (as root):

        $ sudo yum install audit-libs-devel augeas avahi-devel dbus-devel \
               device-mapper-devel ebtables fuse-devel git glusterfs-api-devel \
               glusterfs-devel gnutls-devel libattr-devel libblkid-devel \
               libcap-ng-devel libcurl-devel libnl3-devel libnl-devel \
               libpcap-devel libpciaccess-devel librados2-devel librbd1-devel \
               libselinux-devel libtasn1-devel libvirt-gconfig libvirt-glib \
               libvirt-gobject libvirt-python libxml2-devel ncurses-devel \
               netcf-devel numactl-devel numad parted-devel perl polkit-devel \
               python-requests qemu-img qemu-kvm readline-devel sanlock-devel \
               scrub systemd-devel xhtml1-dtds yajl-devel

1. Download the latest libvirt source RPM and rebuild it:

        $ wget ftp://libvirt.org/libvirt/libvirt-1.2.20-1.fc22.src.rpm
        $ rpmbuild --rebuild libvirt-1.2.20-1.fc22.src.rpm

   The build process will also perform a number of sanity tests on the libvirt binaries. There should be no failures.

1. The binary RPMs will be created in $HOME/rpmbuild/RPMS/x86_64/. Run `rpm -Uvh` as root to install libvirt (or upgrade it, if there is already an existing installation on the system):

        $ cd $HOME/rpmbuild/RPMS/x86_64
        $ sudo rpm -Uvh libvirt-1.2.20-1.el7.x86_64.rpm \
                        libvirt-client-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-config-network-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-config-nwfilter-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-driver-interface-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-driver-lxc-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-driver-network-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-driver-nodedev-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-driver-nwfilter-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-driver-qemu-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-driver-secret-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-driver-storage-1.2.20-1.el7.x86_64.rpm \
                        libvirt-daemon-kvm-1.2.20-1.el7.x86_64.rpm

   Upgrading libvirt should preserve existing libvirt data and configuration files. You may want to examine the configuration files and compare them with the versions saved by `rpm` (*.rpmsave), to ensure that their contents are still correct.

1. **(Optional)** Restart the libvirt daemon:

        $ sudo service libvirtd restart

## Building virt-manager

Now that the new version of libvirt has been built and installed, we can build virt-manager.

1. Install prerequisites for virt-manager (as root):

        $ sudo yum groupinstall 'X Window System' 'GNOME'
        $ sudo yum install git libosinfo openssh-askpass python-ipaddr spice-gtk-python spice-gtk3

1. Clone the virt-manager source code from GitHub, and build it:

        $ git clone https://github.com/virt-manager/virt-manager.git
        $ cd virt-manager
        $ python setup.py build
        $ sudo python setup.py install

    This will install the last version of virt-manager from its GitHub repository and all the data files that it needs under /usr/bin/ and /usr/share/.

1. The previous step installed a new GSettings schema named 'org.virt-manager.virt-manager', which must be compiled into the schema cache before use:

        $ sudo glib-compile-schemas --strict /usr/share/glib-2.0/schemas

1. **(Optional)** Run the self-verifying tests that come with virt-manager:

        $ sudo rm -f /tmp/virtinst-.treeinfo.
        $ python setup.py test

   Note that the tests will produce a file named "virtinst-.treeinfo." in the /tmp/ directory. If the file already exists (from a previous test run), it must be deleted before attempting the tests.