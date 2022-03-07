#### Cloud Security: Cloud Access Control

```How would you control access to a cloud network?```

The "Cloud" offers a lot of advantages in today's increasingly connected world. For many companies the cloud allows them to get up and running faster. A good cloud service also can decrease your security risks by handling many complex and easily misconfigured implementation details "behind-the-scenes" with world-class security experts. 

Controlling access is one of the most important elements of a secure (cloud) system. Let's take a look at how to do that using one of the most popular cloud providers: Microsoft Azure.

For our discussion of best practices regarding cloud network access controls we'll use the cloud network pictured in the [README](README.md) as an example. 

In this project we used several access controls, including Network Security Groups, Load Balancers, a Jump Box / Provisioner machine. Our entire access control plan only has 2 points of contact: Azure Cloud Portal and the Jump Box VM. Minimizing the amount of places we need to configure things decreases the likelihood of making mistakes. 

For our example, we use two Network Security Groups (NSG) configured via Azure. `RedTeam-ELKServer-nsg` contains the ELK Server, and exists in the Azure `Central US` location. `RedTeam2.NetworkSecurityGroup` contains the rest of our machines, including the web servers, the load balancer, and the almighty Jump Box. These two have their own virtual networks which are linked so machines can communicate using their private subnet IPs.

The ELK NSG is configured with 2 inbound rules:
- Allow SSH access (port 22) from the Jump Box IP
- Allow access to port 5601 from my home IP to view Kibana details in browser

The RedTeam2 NSG has more complexity:
- Allow SSH from home IP to Jump Box
- Allow SSH from Jump Box to the 3 redundant webservers
- Allow HTTP access (port 80) from my home IP 

Note that accessing the webservers from the jump box actually uses a container *inside* the jumpbox that is running ansible. Therefore the relevant private ssh key is stored in the ansible docker container, and not the host Jump Box VM. 

One advantage of this solution is that only one machine (the Jump Box) in the entire combined two virtual networks provides terminal access to the outside world. The rest of the machines are only configured from this machine, and therefore we've got only one critical entry point to protect: my home network. An attacker would need to be physically near my house, crack my wi-fi password, and have a copy of my SSH private key (only accessible via my physical computer in the house) in order to gain access to the Jump Box. This applies to accessing Kibana as well, but that's a much lower security concern since it has very limited ability to change my system. 

This advantage also highlights a major disadvantage: my own need to be at home in order to access this location. For each new location I travel to I'll need to either update the Azure NSG rules to match my new IP or add a new rule for the new IP; in either case, someone who travels will find this solution unappetizing.

A better solution for someone who frequently is changing their location is a VPN. A VPN solution (like [OpenVPN](https://openvpn.net/) or [WireGuard](https://www.wireguard.com/)) is relatively simple to setup, and once it's ready allows one-click access to users from any location. 

Another disadvantage to a Jump Box is that once an intruder has access to this machine, they've got the virtual "keys to the kingdom". For example, in 2015 the U.S. Office of Personnel Management was successfully attacked via a compromised jump box. This led to a sizeable government data breach[^1].

For further discussion on the advantages and disadvantages of a Jump Box check out [this O'Reilly article](http://radar.oreilly.com/2014/01/is-the-jump-box-obsolete.html) and, of course, the [wikipedia article](https://en.wikipedia.org/wiki/Jump_server) has several more useful references.

[^1]: https://www.wired.com/2016/10/inside-cyberattack-shocked-us-government/