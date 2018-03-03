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
egrep -E '(svm|vmx)' /proc/cpuinfo

grep -E '(vmx|svm)' /proc/cpuinfo
```


## Installing KVM Packages and enable the required serviced

Centos 7 OS:

```sh
sudo yum groupinstall "Virtualization Host" --optional
sudo yum install virt-manager
```

Alternatively, 
```bash
# yum install qemu-kvm libvirt libvirt-python libguestfs-tools virt-install -y

yum install qemu-kvm qemu-img libvirt virt-install libvirt-python virt-manager libvirt-client virt-viewer bridge-utils -y
```

- `qemu-kvm` = QEMU Emulator 
- `qemu-img` = QEMU Disk Image Manager
- `virt-install` = Command line tool to create virtual machines
- `virt-viewer` = Graphical Interface to see virtual machine console
- `virt-manager` = GUI to Manage Virtual Machines
- `libvirt` = provides `libvirtd` daemon to run services that manage virtual machines and controls hypervisor
- `libvirt-client` = Libvirt provides client-side API for accessing servers and also provides the `virsh` utility which provides command line tool to manage virtual machines.



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
sudo systemctl start libvirtd
sudo systemctl enable libvirtd
```
Now start KVM supportable services:

```sh
systemctl enable libvirtd
systemctl start libvirtd


systemctl start libvirt-guests.service
systemctl enable libvirt-guests.service


systemctl status libvirt-guests.service 
systemctl status libvirtd
```

After installing required packages verify KVM module is visible from kernel using below command. Insert KVM module to kernel using `modprobe` command

```sh 
lsmod | grep kvm

modprobe kvm

lsmod | grep kvm
```
Reboot the Server and then try to start virt manager.

## Configuring `polkit`: Authorization Rules

**Advantages of PolicyKit**
PolicyKit allows for more flexible, fine grained access control than just granting access to a named unix group. 

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

Verify the contents again:
```
sudo cat /etc/polkit-1/localauthority/50-org.example-libvirt-remote-access.pkla
```


This will allow any member of the unix group libvirt to manage the virtualisation layer, including remotely through SSH. 

Log out and log back in for the current user to activate the changes made.

### Connection Example
https://wiki.libvirt.org/page/SSHPolicyKitSetup

We have two users in the libvirt group, named **someuser** and **anotheruser**. Using the PolicyKit Local Authority file above, they should now both have access: 
    
(on a server named HOST1)
    groups someuser anotheruser

(from a computer other than HOST1)
    $ virsh -c qemu=ssh://someuser@HOST1/system
    # output
    Welcome to virsh, the virtualization interactive terminal.
    
    Type:  'help' for help with commands
        'quit' to quit
 
    virsh # hostname
     host1.libvirt.org

(from a computer other than HOST1)
    virsh -c qemu=ssh://anotheruser@HOST1/system
    # output
    Welcome to virsh, the virtualization interactive terminal.
    
    Type:  'help' for help with commands
        'quit' to quit
 
    virsh # hostname
     host1.libvirt.org

### Multiple groups
https://wiki.libvirt.org/page/SSHPolicyKitSetup
Multiple entries can be given on the Identity line. They need to be separated by a semi-colon ";". 

```
[Remote libvirt SSH access]
 Identity=unix-group:group_name1;unix-group:group_name2;unix-group:group_name3
 Action=org.libvirt.unix.manage
 ResultAny=yes
 ResultInactive=yes
 ResultActive=yes
```

### Configuration for Individual Users
Configuring PolicyKit for individual user access is almost identical to the group approach above. The difference is the Identity line in the PolicyKit Local Authority file.

"unix-user" is used instead of "unix-group". 

```
[Remote libvirt SSH access]
 Identity=unix-user:user_name
 Action=org.libvirt.unix.manage
 ResultAny=yes
 ResultInactive=yes
 ResultActive=yes
```

Multiple user names can be given. They need to be separated by a semi-colon ";". 
```
[Remote libvirt SSH access]
 Identity=unix-user:user_name1;unix-user:user_name2;unix-user:user_name3
 Action=org.libvirt.unix.manage
 ResultAny=yes
 ResultInactive=yes
 ResultActive=yes
```

### Configuration for both group and user access

Access can be granted to both groups and individual users at the same time. This is done by using multiple entries on the Identity line of the PolicyKit Local Authority file: 
```
[Remote libvirt SSH access]
 Identity=unix-group:group_name1;unix-user:user_name1;unix-user:user_name2;unix-group:group_name2
 Action=org.libvirt.unix.manage
 ResultAny=yes
 ResultInactive=yes
 ResultActive=yes
```

## GUI Management

Use the app `virt-manager` to manage your KVM installation through the GUI.

Start Virt-Manager 
By Default virtual machines supportable files are going to store in `/var/lib/libvirt/images/` make sure before start virt-manager you have enough space to deploy /create virtual machine. 

`virt-manager` we can either from command line or GUI ( install KVM Hypervisor ) 

From GUI Click on Applications –> System Tools –> Virtual Machine Manager



## CLI Management

The environment variable `LIBVIRT_DEFAULT_URI` can be used to specify the default `libvirt` instance to connect to.

```sh
export LIBVIRT_DEFAULT_URI=qemu:///system
```

Use `virt-install` to install VMs, use `virsh` to manage them. I almost never use `virt-install` but very frequently use `virsh`.

```sh

man virt-install

virt-install --name=microRHEL7 \
    --ram=1024 \
    --vcpus=1 \
    --cdrom=/var/lib/libvirt/images/rhel-server-7.3-x86_64-dvd.iso \
    --os-type=linux \
    --os-path=/var/lib/libvirt/images/rhel7.dsk,size=20
```

```sh

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
BOOTPROTO=none
NAME=eth1
ONBOOT=yes
HWADDR=00:0c:29:44:3d:2d
BRIDGE=virbr0
```
### Create a new interface config file for `vibr0`

Please note `vibr0` is the virtual bridge interface to be used in the VMs

```
cat /etc/sysconfig/network-scripts/ifcfg-virbr0
```

```
TYPE=Ethernet
DEVICE=vibr0
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.1.140
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
```

### Enable the IPv4 forwarding

Add `net.ipv4.ip_forward = 1` entry in `/usr/lib/sysctl.d/60-libvirtd.conf` and load the file

```
/sbin/sysctl -w /usr/lib/sysctl.d/60-libvirtd.conf

/sbin/sysctl net.ipv4.ip_forward

/sbin/sysctl -p /usr/lib/sysctl.d/60-libvirtd.conf
```


### Update the Network Configuration / .DHCP range for the virbr0 interface

```
virsh net-edit default
```

Example config in setup rules are 

```xml
<network>
    <name>default</name>
    <uuid> ... </uuid>
    <forward mode='nat' />
    <bridge name='virbr0' stp='on' delay='0' />
    <mac address='52:54:00:bd:b0:b7' />
    <ip address='102.168.1.140' netmask='255.255.255.0'>             <<< IP of eth1, used as bridged interfaces
        <dhcp>
            <range start='192.168.1.1' end='192.168.1.254' />       
        </dhcp>
    </ip>
</network>
```

### Add a network routing rule for Guests VMs connectivity

```
cat /etc/sysconfig/network-scripts/route-virbr0
```

"192.168.1.0/24 via 192.168.1.140 dev virbr0"

Restart the KVM Host

## Then Create a Guest VM
```
virt-install \
    --network bridge:virbr0 \
    --name testvm1 \
    --os-variant=centos7.0 \
    --ram=1024 \
    --vcpus=1 \
    --disk path=/kvmstore/testvm1.img,size=4 \
    --graphics none \
    --location=/osmedia/CentOS-7-x86_64-DVD-1511.iso \
    --extra-args="console=tty0 console=ttyS0" \
    --check_all=off
```




### CLI Management

The environment variable `LIBVIRT_DEFAULT_URI` can be used to specify the default `libvirt` instance to connect to.

```sh
export LIBVIRT_DEFAULT_URI=qemu:///system
```

Use `virt-install` to install VMs, use `virsh` to manage them. I almost never use `virt-install` but very frequently use `virsh`.

```sh

man virt-install

virt-install --name=microRHEL7 \
    --ram=1024 \
    --vcpus=1 \
    --cdrom=/var/lib/libvirt/images/rhel-server-7.3-x86_64-dvd.iso \
    --os-type=linux \
    --os-path=/var/lib/libvirt/images/rhel7.dsk,size=20
```

```sh

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


## Reference

1. [HowTo Install KVM Hypervisor RHEL 7 Kernel-Based Virtual Machine](https://arkit.co.in/howto-install-kvm-hypervisor-rhel-7/)

2. https://www.centos.org/forums/viewtopic.php?t=57083
















