
# Guest VMs

## Sample 1
```

virt-install --connect qemu:///system \
 -n vmwin7 \
 -r 512 \
 --vcpus=1 \
 --disk path=/vm/vmwin7.img,size=10 \
 --graphics vnc,listen=0.0.0.0 \
 --noautoconsole \
 --os-type windows \
 --os-variant win7 \
 --accelerate \
 --network=bridge:virbr0 \
 --hvm \
 --cdrom /vm/03_Installation-Medium/X17-58997.iso

```

`–connect qemu:///system` : connect to KVM on the local system, we could also connect to another KVM-host and define our new VM there

`-n vmwin7` : name of the new VM: vmwin7

`-r 512` : amount of memory for the VM: 0.5GB

`–vcpus=1` : amount of virtual CPU’s for the VM: 1

`–disk path=/vm/vmwin7.img,size=10` : where to store the virtual disk image of the VM and the size: 10GB

Default `path=/var/lib/libvirt/images/`




`–graphics vnc,listen=0.0.0.0` : how to display the VM’s console: via VNC accessible from outside

`–noautoconsole` : do not automatically connect to the console

`–os-type windows –os-variant win7` : type of guest OS (from the list given above)

`–accelerate` : use KVM HW-acceleration

`–network=bridge:virbr0` : network bridge to use

`–hvm` : full virtualisation

`–cdrom /vm/03_Installation-Medium/X17-58997.iso` : location of the installation ISO

## Spin More VMs

### vm001
```
virt-install --connect qemu:///system \
 -n vm001 \
 -r 512 \
 --vcpus=1 \
 --disk path=/vm/vm001.img,size=10 \
 --graphics vnc,listen=0.0.0.0 \
 --noautoconsole \
 --os-type linux \
 --os-variant centos7.0 \
 --accelerate \
 --network=bridge:virbr0 \
 --hvm \
 --cdrom /vm/03_Installation-Medium/CentOS-7-x86_64-DVD-1503-01.iso
```

### vm002
```
virt-install --connect qemu:///system \
 -n vm002 \
 -r 512 \
 --vcpus=1 \
 --disk path=/vm/vm002.img,size=10 \
 --graphics vnc,listen=0.0.0.0 \
 --noautoconsole \
 --os-type linux \
 --os-variant centos7.0 \
 --accelerate \
 --network=bridge:virbr0 \
 --hvm \
 --cdrom /vm/03_Installation-Medium/CentOS-7-x86_64-DVD-1503-01.iso
```

### vm003
```
virt-install --connect qemu:///system \
 -n vm003 \
 -r 512 \
 --vcpus=1 \
 --disk path=/vm/vm003.img,size=10 \
 --graphics vnc,listen=0.0.0.0 \
 --noautoconsole \
 --os-type linux \
 --os-variant centos7.0 \
 --accelerate \
 --network=bridge:virbr0 \
 --hvm \
 --cdrom /vm/03_Installation-Medium/CentOS-7-x86_64-DVD-1503-01.iso
```

### vm004
```
virt-install --connect qemu:///system \
 -n vm004 \
 -r 512 \
 --vcpus=1 \
 --disk path=/vm/vm004.img,size=10 \
 --graphics vnc,listen=0.0.0.0 \
 --noautoconsole \
 --os-type linux \
 --os-variant centos7.0 \
 --accelerate \
 --network=bridge:virbr0 \
 --hvm \
 --cdrom /vm/03_Installation-Medium/CentOS-7-x86_64-DVD-1503-01.iso
```


