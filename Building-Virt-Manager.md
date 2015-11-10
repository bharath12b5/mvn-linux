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

## Managing guests on a remote KVM hypervisor running on IBM z Systems

1. **(Optional)** virt-manager will invoke the [gnome-ssh-askpass](http://manpages.ubuntu.com/manpages/hardy/man1/gnome-ssh-askpass.1.html) utility to prompt for a SSH passphrase for the remote system interactively. Before starting virt-manager, [generate a pair of SSH keys](https://help.github.com/articles/generating-ssh-keys/) and copy the public key into the file `~root/.ssh/authorized_keys` on the KVM hypervisor. You can use a key without a passphrase to streamline the login process, but doing so has security implications and is not recommended for production systems.

   If you do not use SSH public key authentication, SSH will prompt for a password in the terminal. For this to work, you must invoke virt-manager with the `--no-fork` option.

1. Launch virt-manager in a terminal or open it through the application menu:

        $ virt-manager [--no-fork]

1. To create a connection to the KVM hypervisor, open the File menu and choose "Add Connection". Select the option to "Connect to remote host", choose the "SSH" connection method, fill out the username and hostname fields, and then click "Connect".

   ![Adding a new connection](docs/wiki/img/virt-manager-01-add-connection.png)

   After connecting to the hypervisor, all available guests will be listed in the virt-manager window.

1. To install a Linux guest from an ISO image on the KVM server, you need to mount the ISO image, extract the kernel and initrd files from it, and place them in a temporary location. The following example is for copying the files from a SLES 12 SP3 ISO image.

        # mkdir /tmp/iso-files
        # mount -o ro,loop /<path-to-ISO>/ /mnt
        # cd /mnt/boot/s390x
        # cp ./linux ./initrd /tmp/iso-files
        # cd /tmp/iso-files
        # chmod 755 linux initrd
        # umount /mnt

1. Create a new virtual machine by clicking the "New" button in virt-manager. This will bring up the "New VM" dialog box. Ensure that you are connected to the correct KVM hypervisor as shown in the "Connection" drop-down menu. Select "Local install media (ISO image or CDROM)". Within the architecture options, ensure that "Virt Type" is set to "KVM" and "Machine Type" is set to "s390-ccw-virtio". Click "Forward".

   ![Adding a new virtual machine](docs/wiki/img/virt-manager-02-add-vm.png)

1. Browse and select the ISO image to install from. The OS type and version can be specified if matched with the ISO or left as "Generic". Click "Forward".

1. Select the amount of RAM and CPUs to allocate to the virtual machine. Click "Forward".

1. Enter a name and size for the disk storage for the virtual machine. Click "Forward".

1. Enter a name for the guest VM, and select the checkbox to "Customize configuration before install".

   ![Customizing configuration](docs/wiki/img/virt-manager-3-customize-install.png)

   Click "Finish". This will bring up the VM configuration dialog box. Select "Boot Options" from the menu on the left, then expand the "Direct kernel boot" section and select "Enable direct kernel boot". In the "Kernel path" field, Enter the path to the extracted kernel file, e.g. `/tmp/iso-files/linux`. Similarly, in the "Initrd path" field, enter the path to the extracted initrd file, e.g. `/tmp/iso-files/initrd`. Click "Apply", then click "Begin Installation".

   ![Changing the boot options](docs/wiki/img/virt-manager-4-boot-options.png)

1. The console window to the new VM will open after a brief pause, and the installation of the guest operating system will begin. After completing the installation, the installer will typically reboot the system. The reboot will use the direct kernel boot parameters again instead of booting into the newly installed guest. To change the boot options, shut down the VM by clicking "Shut Down" in the console window, click "Details", navigate to the "Boot Options" menu, and uncheck the "Enable direct kernel boot" option.

1. Restart the virtual machine. The guest operating system should boot up correctly.

## References

* [Creating and Managing Guests with Virt-Manager](https://docs.fedoraproject.org/en-US/Fedora/23/html/Virtualization_Getting_Started_Guide/ch06.html)
* [Managing VMs with the Virtual Machine Manager](http://www.ibm.com/developerworks/cloud/library/cl-managingvms/)
