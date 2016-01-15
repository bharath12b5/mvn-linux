virt-install is a command line tool for creating new KVM , Xen, or Linux container guests using the "libvirt" hypervisor management library. In this instruction we build virt-install from the source code which is contained in the virt-manager package (https://git.fedorahosted.org/git/virt-manager.git).

The test is performed on KVM for IBM z Systems release 1.1.0 GA which is based on RHEL 7. Since virt-install requires root permission to run you need to perform the following steps with root privilege.

## Prerequisites

### Install dependencies.

    $ yum install libvirt, libvirt-python, python-ipaddr, perl-XML-Parser, intltool

### Install libosinfo >= 0.2.10

    $ wget ftp://rpmfind.net/linux/fedora-secondary/development/rawhide/s390x/os/Packages/l/libosinfo-0.2.12-2.fc23.s390x.rpm
    $ rpm -ivh libosinfo-0.2.12-2.fc23.s390x.rpm

## Install the latest virt-manager from source code which contains the patches for s390x and SLES support

    $ git clone https://github.com/virt-manager/virt-manager.git
    $ cd virt-manager
    $ python setup.py install

## Testing

### Test one: Unit test

    $ cd virt-manager
    $ python setup.py test

It should pass all the tests.

### Test two: Installing a guest vm from a local iso image

1. Create a disk image (for storage). The KVM hypervisor supports qcow2. qcow2 images support compression, snapshots and a few other nice things like growing on demand (thin provisioning, sparse file) and a read-only base image. There was a performance overhead but nowadays that is almost negligent. To create an 8 GB qcow2 image:

    ```
    $ qemu-img create -f qcow2 ./name.qcow2 8G
    ```

2. Download an OS image to you local directory, say Fedora-Server-DVD0s390x-22.iso. We have also tested with RHEL 7 and SLES 12.

    **Troubleshooting:**

    You have to have root privileges to run virt-install. And make sure name.qcow2 and *.iso files are accessible by user qemu.

3. As a normal user, run virt-install following the example below:

    ```
    $ sudo virt-install --name fedora22 --ram 1024 --disk path=/path_to_storage_img/name.qcow2,size=8 --vcpus 1 --os-type linux --os-variant generic --network bridge=virbr0 --graphics none --location /path_to_os_img/Fedora-Server-DVD-s390x-22.iso
    ```

    Then you can choose the text mode for basic installation configuration, such as time zone, root password, user account/password, etc.

4. To manage VMs, use virsh command - some examples noted here: https://help.ubuntu.com/community/KVM/Virsh

### Test three: Using virt-clone

1. List all the vm created

    ```
    $ sudo virsh list --all
    ```

2. Clone a vm

    ```
    $ sudo virt-clone --original origianl_vm_name --name new_vm_name --file /absolute_path/storage_file
    ```

### Test four (Optional): Using MacVTap for networking

MacVTap is a new device driver meant to simplify virtualized bridged networking. It replaces the combination of the tun/tap and bridge drivers with a single module based on the macvlan device driver. A MacVTap endpoint is a character device that largely follows the tun/tap ioctl interface and can be used directly by kvm/qemu and other hypervisors that support the tun/tap interface. The endpoint extends an existing network interface, the lower device, and has its own mac address on the same ethernet segment. Typically, this is used to make both the guest and the host show up directly on the switch that the host is connected to.

Assume the MacVTap is set up on interface guest81, you can configure virt-install to install a VM using MacVTap driver as follows:

    ```
    $ sudo virt-install --name vm_name --ram 1024 --disk path=/absolute path/vm_name.qcow2,size=8 --vcpus 1 --os-type linux --os-variant generic --network type=direct,source=guest81,source_mode=bridge --graphics none --location /absolute_path/os_img.iso
    ```