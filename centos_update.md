# CentOS Update

## Steps

Check the current version of your release:
```
cat /etc/redhat-release
```

**Backup Important data and directory**


Lookup what updates are available:
```
yum check-update
```

Issue the following command to install the updates:
```
yum clean all
yum update
```

Reboot after complete:
```
reboot
```

Verify CentOS version:
```
cat /etc/redhat-release
```

