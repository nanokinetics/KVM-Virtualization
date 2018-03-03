# Open VNC Ports in Firewall

**To add a source (ex: 192.168.2.0/24) to a zone (here trusted) permanently, type:**
```
firewall-cmd --permanent --zone=trusted --add-source=192.168.2.0/24
firewall-cmd --reload
```

## Connect via VNC to Virtual Machine

**Getting the Screen-Number**:
```
virsh vncdisplay vmwin7
netstat -tln | grep :59
```

```
192.168.2.22:5900
kvmhost:5900
spice://kvmhost:5900
```

```

virt-install --connect qemu:///system \
 -n vmwin7 \
 -r 512 \
 --vcpus=1 \
 --disk path=/vm/vmwin7.img,size=10 \
 --graphics spice,listen=0.0.0.0 \
 --noautoconsole \
 --os-type windows \
 --os-variant win7 \
 --accelerate \
 --network=bridge:virbr0 \
 --hvm \
 --cdrom /vm/03_Installation-Medium/X17-58997.iso
```

## Remove Virtual Machine from Virtual Manager and Harddisk

```
virsh destroy vmwin7
virsh undefine vmwin7
rm -rf /vm/vmwin.img
```
