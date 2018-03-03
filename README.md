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

**Host**:

**Guest**: 

**Hypervisor**:

<hr>
File: `host`
```
192.168.2.22 kvmhost kvmhost.nanokinetics.io
```

Add SSH Public Keys: 
```
# ssh into kvmhost from a remote system
ssh root@kvmhost
mkdir -p ~/.ssh
chmod 600 .ssh
exit

# from remote system
cat ~/.ssh/*.pub | ssh root@kvmhost "cat >> ~/.ssh/authorized_keys"
```
<hr>
## Check if VT (i.e. Virtualization Technology) is enabled/support on the KVM host

```sh
egrep -E '(svm|vmx)' /proc/cpuinfo

grep -E '(vmx|svm)' /proc/cpuinfo
```

## Disable and stop NetworkManager
**NetworkManager** is known to caus eproblems wen working with Linux Bridge, so better to disable it:
```
sudo systemctl stop NetworkManager
sudo systemctl disable NetworkManager
```

## Installing KVM Packages and enable the required serviced

Centos 7 OS:

```sh
sudo yum groupinstall "Virtualization Host"
sudo yum install virt-manager
```

Alternatively, PREFERRED  :heavy_check_mark:
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

Enable and start KVM supportable services:

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

# Add the User to the group 'virt', this lets regular user to let him launch `virt-manager` 
sudo usermod -aG virt <UserName>

sudo mkdir -p /etc/polkit-1/localauthority/50-local.d/
```

**METHOD-1**: :OK:

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

**METHOD-2**: :warn:

We can set policy (policy-kit) rules for KVM.

Edit file **49-polkit-pkla-compat.rules**:

```
vim /etc/polkit-1/rules.d/49-polkit-pkla-compat.rules
```
add the following at the bottom:
```
polkit.addRule(function(action, subject) {
    if (action.id == "org.libvirt.unix.manage"
        && subject.local
        && subject.active
        && subject.isInGroup("<GROUPNAME>")) {
            return polkit.Result.YES;
        }
});
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

REFER: `man nmcli-examples`


### Update the Base Interface File (BIF).
In this example, `eth1` is the interface connected to internet and will be used as Bridge Interface for the VMs.

```sh
cat /etc/sysconfig/network-scripts/ifcfg-enp4s0f1
```

```
TYPE=Ethernet
DEVICE=enp4s0f1
# BOOTPROTO=none
NAME=enp4s0f1
ONBOOT=yes
HWADDR=00:0c:29:44:3d:2d
BRIDGE=virbr0

DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
UUID:
PEERDNS=yes
PEERROUTES=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
```
### Create a new interface config file for `virbr0`

Please note `virbr0` is the virtual bridge interface to be used in the VMs

```
cat /etc/sysconfig/network-scripts/ifcfg-virbr0-enp4s0f1
```

```
TYPE=Bridge
BOOTPROTO=static
# DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
# IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=virbr0
UUID=3e775e0c-9164-41a6-a7e6-8a58a2a05396
DEVICE=virbr0
ONBOOT=yes
IPADDR=192.168.188.30
NETMASK=255.255.255.0
GATEWAY=192.168.188.1
DNS1=192.168.188.1
```

### Enable the IPv4 forwarding

Add `net.ipv4.ip_forward = 1` entry in `/usr/lib/sysctl.d/60-libvirtd.conf` and load the file


```
echo "net.ipv4.ip_forward = 1"|sudo tee /etc/sysctl.d/99-ipforward.conf
sysctl -p /etc/sysctl.d/99-ipforward.conf
ifconfig -a
```



```
# DONT USE THIS
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

### Finish and check the KVM installation
```
lsmod | grep kvm
ip a show virbr0
virsh -c qemu:///system list
```


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
Please Note: 

<table>
    <tbody>
    <tr>
        <td>--network bridge:virbr0</td>
        <td>is the underlying network interface for guest VMs NIC</td>
    </tr>
        <tr>
        <td>--name testvm1</td>
        <td>is the name of the bost VM/container</td>
    </tr>
    <tr>
        <td>--os-variant=centos7.0</td>
        <td>this is the label for CentOS 7 linux variant or distribution</td>
    </tr>
    <tr>
        <td>--ram=1024</td>
        <td>this will allocate 1G RAM to the guest VM</td>
    </tr>
    <tr>
        <td>--vcpus=1</td>
        <td>this will allocate 01 vCPU to guest VM</td>
    </tr>
    <tr>
        <td>--disk_path=/kvmstore/testvm1,size=4</td>
        <td>this creates /kvmstore/testvm1.img as OS store file for the guest VM</td>
    </tr>
    <tr>
        <td>--graphics none</td>
        <td>this set no graphical user interface, installation will go command line</td>
    </tr>
    <tr>
        <td>--location=/osmedia/CentOS-7-x86_64-DVD-1511.iso</td>
        <td>location of OS media to be used for the OS installation</td>
    </tr>
    <tr>
        <td>--extra-args="console=tty0 console=ttyS0, 115200"</td>
        <td>Console connectivity options</td>
    </tr>
    <tr>
        <td>--check all=off</td>
        <td>this is optional, but used for overriding things</td>
    </tr>
    </tbody>
</table>



### CLI Management

The environment variable `LIBVIRT_DEFAULT_URI` can be used to specify the default `libvirt` instance to connect to.

```sh
export LIBVIRT_DEFAULT_URI=qemu:///system
```

Use `virt-install` to install VMs, use `virsh` to manage them. I almost never use `virt-install` but very frequently use `virsh`.

```
virt-install --help
```

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
## SELinux Virtualbox Image Storage Directory Change:

```
mkdir /vm
semanage fcontext -a -t virt_image_t "/vm(/.*)?"
restorecon -R /vm
```



## Common KVM Management Tasks

List all VMs on a host, in any state
```
virsh list --all
```

To display the VM information/configuration
```
virsh dominfo <vm_name>
```

Stop a VM
```
virsh shutdown <vm_name>
```

Start a VM
```
virsh start <vm_name>
```

Enabling AutoStart of VM
```
virsh autostart <vm_name>
```

Taking console of the VM, from the KVM Host
```
virsh console <vm_name>
```

Deleting Guest VM
```
virsh shutdown <vm_name>
virsh undefine <vm_name>
virsh destroy <vm_name>
```

Cloning a VM (Suspend the Source VM first, once cloning finishes resume it.)
```
virsh suspend <source_vm>
virsh-clone --connect qemu:///system --original <source_vm> --name <clone_vm>
virsh resume <source_name>
```
































## Reference

1. [HowTo Install KVM Hypervisor RHEL 7 Kernel-Based Virtual Machine](https://arkit.co.in/howto-install-kvm-hypervisor-rhel-7/)

2. https://www.centos.org/forums/viewtopic.php?t=57083
















