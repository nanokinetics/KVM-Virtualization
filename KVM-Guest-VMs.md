
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

