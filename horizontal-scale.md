# How to scale a server horizontally? or in other words Scale out?

## Availability set

Pre-requisite for High Availability
https://docs.microsoft.com/en-us/azure/virtual-machines/windows/manage-availability

Ideally, you have one for your Web Servers and another for the Database (not the same one for both!).

A VM can only be placed in an Availability Set during its creation, that said, if you don't have one configured on your VMs at the moment, you will need to redeploy. No worries, its easy! Here's the process:

https://docs.microsoft.com/en-us/azure/virtual-machines/windows/change-availability-set

## Load Balancer

Now that you have the AS configured, you can deploy and configure the Load Balancer.
https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-get-started-ilb-arm-portal

For the Web Servers, you may want to configure External Load Balancers instead. Make sure to also look at the Application Gateway capabilities.


## MongoDB
Bear in mind that to achieve High Availability with MongoDB, you need at least 3 nodes and there's some additional configuration you have to perform to be able to failover the servers.

https://docs.mongodb.com/manual/administration/replica-set-deployment/


## Parse Server
As far I know there are no special requirements to be able to scale the Parse Server, but you can also use the process described in this article:
http://blog.kontena.io/how-to-install-and-run-private-parse-server-in-production/





