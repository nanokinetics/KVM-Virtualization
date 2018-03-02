Major changes CentOS 7.4:

    CentOS can report bugs directly to bugs.centos.org via new abrt release.
    OpenSSL now supports Datagram Transport Layer Security (DTLS).
    Amazon ENA – Elastic Network Adapter drivers have been added to the kernel
    Technology Preview: Btrfs, OverlayFS, CephFS, the Cisco VIC and usNIC kernel driver and much more.


https://www.dynu.com/Resources/Tutorials/DynamicDNS/How-to-access-CentOS-Ubuntu-Linux-Machine-remotely

# Access CentOS/Ubuntu Linux Machine Remotely

## Step 1: Enable SSH on Linux Host

Facts:
1. 'ssh' runs on port 22 by default
2. 

### Ubuntu 16.04 LTS
1. Install OpenSSH server: `sudo apt-get install openssh-server`

2. Check SSH service status: `sudo service ssh status`

3. Edit SSH config file: `sudo vi /etc/ssh/sshd_config`

4. Reload SSH: `sudo service ssh restart`

### CentOS/RHEL 7
1. Install OpenSSH server: `yum install openssh`

2. Check SSH service status: `systemctl status sshd`

3. Start SSH: `systemctl start sshd`

4. Edit SSH config file: `sudo vi /etc/ssh/sshd_config`

5. Reload SSH: `systemctl restart sshd.service`



## Step 2: Setup Port Forwarding (Port Translation) in the router

Please log into the router interface (generally at http://192.168.1.1 or http://192.168.0.1) and go into the 'Port Forwarding' section. Add a new 'Port Forwarding' rule for TCP port 22 to be forwarded to the internal IP of your Linux machine. The machine will be assigned an internal IP address even if you are running it as a Virtual Machine. To get the internal IP address, you may type ifconfig -i. It is usually in the form of "192.168.0.**". 

![](https://www.dynu.com/content/images/content/Tutorials/DynamicDNS/Introduction/Check-Internal-IP.png)

NOTE: If you need to connect to several Linux machines behind the same router, you should set up port forwarding for all these machines. Let's suppose Linux machine 1 has an internal IP 192.168.0.24, machine 2 has an internal IP 192.168.0.46, and machine 3 has an internal IP 192.168.0.58. We can setup different external ports for different machines as shown in the picture below. You will need to change the SSH default port in the Linux machines as well by editing the config file located at /etc/ssh/sshd_config. 

![](https://www.dynu.com/content/images/content/Tutorials/DynamicDNS/Introduction/SSH-PortForwarding.png)

To see if the port forwarding has been setup correctly, you can use our Port Check network tool to see if the corresponding port is open. If you get a "Success" response from the port check, then your network has been correctly set up. 
