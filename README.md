# KVM-Virtualization

## Smart Facts to Remember
- KVM stands for Kernel-based Virtual Machine
- KVM is a type-2 Hypervisor, which means it runs on a host operating system
- Other type-2 hypervisors: VirtualBox and Hyper-V
- In contrast, type-1 hypervisors run on the bare metal and don't need a host operating system.
- Type-2 Hypervisors include Xen and VMware ESX.

### Use-Case
Using KVM we can run multiple virtual machines within a server. It does support multiple operating systems ranging from Windows, SUSE, CentOS, Ubuntu, ...

### KVM Hypervisor Advantages
- Open Source Solution (No vendor lock-in)
- Cross platform support
- Affordable
- High performance w.r.t workloads
- Secure (SELinux support)


## Check if VT (i.e. Virtualization Technology) is enabled/support on the KVM host

```sh
grep -E '(svm|vmx)' /proc/cpuinfo
```


## Installing KVM Packages and enable the required serviced

Centos 7 OS:

```sh
sudo yum groupinstall "Virtualization Host"
sudo yum install virt-manager
```

Alternatively, 
```bash
# yum install qemu-kvm libvirt libvirt-python libguestfs-tools virt-install -y

yum install qemu-kvm qemu-img libvirt virt-install libvirt-python virt-manager virt-install libvirt-client virt-viewer -y
```

- `qemu-kvm` = QEMU Emulator 
- `qemu-img` = QEMU Disk Image Manager
- `virt-viewer` = Graphical Interface to see virtual machine console
- `virt-manager` = GUI to Manage Virtual Machines
- `libvirt` = libvirtd daemon to run services
- `libvirt-client` = Libvirt client packages

After installing required packages verify KVM module is visible from kernel using below command. Insert KVM module to kernel using `modprobe` command

```sh 
lsmod | grep kvm

modprobe kvm

lsmod | grep kvm
```

Now start KVM supportable services:

```sh
systemctl enable libvirt-guests.service

systemctl enable libvirtd

systemctl start libvirt-guests.service

systemctl start libvirtd

systemctl status libvirt-guests.service 

systemctl status libvirtd
```

**NOTE**: 
Note: virt-Manager and virt-viewer required Graphical User Interface to launch virtual machine manager

If you have installed only minimal installation Operating system then you must install GUI

```sh
yum install "@X Window System" xorg-x11-xauth xorg-x11-fonts-* xorg-x11-utils -y
```

Or

```sh
yum groupinstall "Server with GUI"
```


Status check:
```sh
sudo systemctl status libvirtd
```
By default, it is "active(running)". 

If not...then,

Start/enable the services:
```sh
sudo systemctl enable libvirtd
sudo systemctl start libvirtd
```


## Configuring `polkit`

The following allows a non-root user access to manage KVM guests

See: [Reference](http://www.linuxsysadmintutorials.com/configure-polkit-to-run-virsh-as-a-normal-user/)

```sh
# Create a group 'virt'
sudo groupadd virt

# Add the User to the group 'virt' 
sudo usermod -aG virt <UserName>

sudo mkdir -p /etc/polkit-1/localauthority/50-local.d/
```

Create a file `/etc/polkit-1/localauthority/50-local.d/50-org.example.libvirt-access.pkla` with contents:

```
[libvirt Admin Access]
Identity=unix-group:virt
Action=org.libvirt.unix.manage
ResultAny=yes
ResultInActive=yes
ResultActive=yes
```

Log out and log back in for the current user to activate the changes made.

## GUI Management

Use the app `virt-manager` to manage your KVM installation through the GUI.

Start Virt-Manager 
By Default virtual machines supportable files are going to store in `/var/lib/libvirt/images/` make sure before start virt-manager you have enough space to deploy /create virtual machine. 

virt-manager we can either from command line or GUI ( install KVM Hypervisor ) 

From GUI Click on Applications –> System Tools –> Virtual Machine Manager



## CLI Management

The environment variable `LIBVIRT_DEFAULT_URI` can be used to specify the default `libvirt` instance to connect to.

```sh
export LIBVIRT_DEFAULT_URI=qemu:///system
```

Use `virt-install` to install VMs, use `virsh` to manage them. I almost never use `virt-install` but very frequently use `virsh`.

```sh

man virt-install
# List all guests/domains on system, including those that are powered off
virsh list --all
# Turn on a guest
virsh start <domain>
# Shutdown the guest
virsh shutdown <domain>
# Pull the plug
virsh destroy <domain>
# RTFM
man virsh

```

## VirtIO for Windows

In general, the virtio devices (NIC, Storage) provide best performance. Most recent Linux kernels (since version?) have these drivers included. Windows guests won't.

You'll need to mount an ISO containing the virtio drivers during Windows guest installation. See: https://fedoraproject.org/wiki/Windows_Virtio_Drivers (This also includes the memory balloon drivers).

    sudo wget https://fedorapeople.org/groups/virt/virtio-win/virtio-win.repo -O /etc/yum.repos.d/virtio-win.repo
    sudo dnf install virtio-win
    sudo yum install virtio-win

During installation, mount the iso `/usr/share/virtio-win/virtio-win.iso` and load the `viostor`, `netkvm`, and `balloon` drivers.


## Configure the Networking for KVM

### Update the Base Interface File (BIF).
In this example, `eth1` is the interface connected to internet and will be used as Bridge Interface for the VMs.

```sh
cat /etc/sysconfig/network-scripts/ifcfg-eth1
```

```
TYPE=Ethernet
NAME=eth1
ONBOOT=yes
HWADDR=00:0c:29:44:3d:2d
BRIDGE=virbr0
```





















