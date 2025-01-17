## Network > VPC > Console Guide

This document describes what you need to do when working with VPCs in the console.

## VPC

Since a VPC can have multiple subnets, a sufficiently large network must be configured when divided subnets are used. A VPC network can be described by using [CIDR notation](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing). All VPCs must be located in the three address areas shown below, where a [private network](https://en.wikipedia.org/wiki/Private_network) can be configured, and link-local addresses cannot be used. In addition, you must specify a network area that is larger than 24bit-256.

### Private Network 

RFC1918 | IP Address Area | Available Number of Addresses 
-------- | ---------- | -----------------
24bit block | 10.0.0.0/8 | 16,777,216
20bit block | 172.16.0.0/12 | 1,047,576
16bit block | 192.168.0.0/16 | 65,536

### Link-Local Addresses 

Cannot use 65,536 IP addresses that are included to 169.254.0.0/16. 

### Examples 

Example | Availability 
-------- | ---------- 
10.0.0.0/8 | Available. 
10.0.0.0/16 | Available. 
10.0.0.0/24 | Available. 
10.0.0.0/28 | Unavailable. Range is too short. 
172.16.0.0/16 | Available. 
172.16.0.0/8 | Unavailable. Out of available range. 
192.168.0.0/16 | Available. Specified as default range. 
192.168.0.0/24 | Available. 
192.253.0.0/24 | Unavailable. Out of available range. 

<br>


By using initial compute and network products, items as below are to be automatically configured. 

Item | Name | Summary 
-------- | ---------- | --------------
VPC | Default Network | Creates one VPC within the range of 192.168.0.0/16. 
Subnet | Default Network | Creates one subnet within the range of 192.168.0.0/24. 
Routing Table | vpc-[id] | Creates one routing table named after a part of VPC ID. 
Internet Gateway | ig-[id] | Creates one internet gateway named after a part of routing table ID. 
Security Group | default | Creates a security group named default. 

To add VPC, not as an initial configuration, configure items as below: 

Item | Name | Summary 
-------- | ---------- | --------------
VPC | As specified | Creates one VPC within the specified range. 
Subnet | - | Not created. 
Routing Table | vpc-[id] | Creates one routing table named after a part of VPC ID. 
Internet Gateway | - | Not created: create additionally and connect. 
Security Group | - | Not created additionally. 

Here is the quota for VPC and each item.  

Item | Max 
-------- | ---------- 
VPC | 3
Subnet | 10 per VPC 
Internet Gateway | 3 
Floating IP | Unlimited 
Routing Table | 10 per VPC 
Route | 10 per Routing Table 
Peering | Unlimited 
NAT gateway | 3


> [Note]
VPCs can be deleted only when subnets are deleted altogether, and in such case, delete along with subnets, routing tables, and internet gateways. 

* VPC is safe from traffic because it is completely isolated from other VPCs.

* Direct access of VPC from the internet is unavailable as it is a private network.  

* All elements within VPC cannot use VLAN. 

* For those traffic beyond a region, local communication is not provided.  

* All instances within a VPC cannot access the internet without internet gateway. 

* Excessively transferred "Broadcast, Multicast, Unknown Unicast" may be blocked without prior notice. 


## Subnets 

A VPC can be subdivided into subnets that are composed of multiple networks. However, a subnet must be included within the range of a VPC address, with its length the same or shorter. For example, in the case of 192.168.0.0/16, a total of 65536 IP addresses are available between 192.168.0.0 and 192.168.255.255. In addition, the smallest subnet is 28 bits and any configuration cannot be smaller than that. Subnets also adopt CIDR display, just like VPC.  

When a subnet is created, it is automatically associated with the default routing table included in the VPC. At this time, the gateway IP is assigned automatically.

> [Note]
To delete a subnet, it should be empty with no instances or load balancers included. Also, make sure that the routing table which the subnet is associated with does not have routes to that subnet.

* Assigns one IP address to a newly-created instance from a designated subnet (called a fixed IP).

* Applies the IP address to the instance when it is booted, via DHCP.  

* Cannot modify the address range of a subnet. 

* Cannot redundantly create or overlap the range for different subnets within a same VPC.

* The range of subnets may be redundant or wrapped over for different VPCs. 

* MAC addresses that are not assigned to an instance may be blocked from the network. Therefore, a VPN service running on instances may not work. 

* When multiple subnets are connected to an instance, an appropriate routing setting is required within OS of the instance. 

* Two subnets within a same VPC are not completely isolated. Apply security groups to protect instances. 

* Subnets support local communication through different areas of availability. Local communication shall not be charged.  

By using a subnet's "Static Route" setting, it is possible to pass the routing rules that instances in the subnet will set in the routing table within the instances at boot time.

* The routing rules registered in "Static Route" are sent by being included in the "classless-static-routes" option of the response to the DHCP request requested by the instance. The DHCP client running in the instance registers the content of this option in the routing table.

* The method of reflecting the "classless-static-routes" option varies depending on the OS type, distribution, or DHCP client version running on the instance. However, it is generally reflected when the DHCP client is first started, such as instance booting, and is not reflected when renewing the DHCP lease. Therefore, when you edit a subnet's "Static Route", the change may not take effect immediately on the running instances, so it is recommended to reboot the running target instances if possible.

* A "Static Route" consists of the destination CIDR of the packet to be routed and the gateway information to forward the target packet to.<br>If you create a static route with a CIDR of "0.0.0.0/0", you can change the default gateway of the instance to an IP other than the gateway of the subnet.<br>The gateway is entered as text as opposed to a "Route" in the routing table, and you can also specify an IP that is not yet assigned within the subnet.

## Internet Gateway

An internet gateway can be associated with a routing table. VPCs consisting of private networks cannot be connected externally, and may access the internet through internet gateway. Each instance, to be connected to an internet, must set "default gateway" as the gateway address of the subnet, and NHN Cloud does the job automatically. To create an internet gateway, an external network is required and NHN Cloud is operating only one "public_network" as of now.  

* An internet gateway address is automatically assigned when an instance is created or VPC requires an internet access, and it cannot be modified. 

* Cannot access an instance with an internet gateway address. 

* Blocks all traffic inflow via internet gateway address.

* Charges by usage, when an instance connected to the internet triggers traffic to the internet direction.

* Do not charge for local communication between instances. 

### Guide for Restarting Internet Gateways for Server Maintenance 

NHN Cloud updates software of the Internet gateway server on a regular basis to enhance security and stability of its infrastructure services. 

Internet gateways that are running on a target server for maintenance must be restarted and migrated to the server which is completed with maintenance.  

To restart Internet gateways, use the **! Restart** button which is created next to each Internet gateway name. 

Go to the project where your Internet gateway specified as maintenance target is located. 

1. Any Internet gateway that has the **! Restart** button before its name requires maintenance. 
    ![ig-001](http://static.toastoven.net/prod_vpc/ConsoleGuide/ig_planned_migration_guide-en-001.png)
    Put the mouse cursor on the **! Restart** button to find maintenance schedule details. 
    ![ig-002](http://static.toastoven.net/prod_vpc/ConsoleGuide/ig_planned_migration_guide-en-002.png)

2. Select a target Internet gateway and click the **! Restart** button next to its name. 
    It is advised to perform maintenance during time when impact on service is limited, since instances of the Internet gateway are disconnected from the Internet until restarting is completed. 
    However, instances that are associated with floating IPs are not influenced by restarting of Internet gateways. 

3. Click **OK** onto the window asking of restarting Internet gateway. 
    ![ig-003](http://static.toastoven.net/prod_vpc/ConsoleGuide/ig_planned_migration_guide-en-003.png)

4. Wait until the Internet gateway status turns green and the **! Restart** button disappears.
    If the status does not change or the **! Restart** button remains, press 'Refresh'.  
    ![ig-004](http://static.toastoven.net/prod_vpc/ConsoleGuide/ig_planned_migration_guide-en-004.png)

The Internet gateway becomes inoperable while restarting is underway.
Unless restarting Internet gateway is normally completed, it shall be automatically reported to the administrator, and you'll be contacted by NHN Cloud.  


## NAT gateway

This feature is available only in the Pyeongchon region, Korea.
NAT gateway can be set as a gateway from the route setting of a routing table. If NAT gateway is set as the gateway for the routing table, the floating IP of NAT gateway can be switched to source IP to access the Internet.

| Item      | Description                                                         |
| --------|------------------------------------------------------------ |
| Name      | Can specify the name of NAT gateway. |
| VPC      | Specifies the VPC to create NAT gateway. |
| Subnet     | Specifies the subnets associated with a routing table that is associated with the Internet gateway among the subnets of the selected VPC. NAT gateway cannot be created with a subnet that does not satisfy the conditions. |
| Floating IP  | Specifies the floating IP to be assigned to NAT gateway. This IP is converted to the source IP when connecting to the Internet. |
| Description      | Can add the description for NAT gateway. |

* The VPC, subnet, and floating IP of an NAT gateway cannot be changed.

* The floating IP configured by NAT gateway cannot be disconnected. It is automatically disconnected when the NAT gateway is deleted.

* Instances cannot be accessed with the NAT gateway address.

* The inbound traffic to the NAT gateway address is entirely blocked.

* A single NAT gateway can be specified as a gateway in multiple routing tables in the same VPC.

* Subnets specified when NAT gateway is configured and routing tables associated with a different subnet can specify NAT gateway as its gateway.

* Network ACL is applied.

* If the gateway is set to 'NAT gateway’ for the target CIDR to route without IP Prefix 0 (/0) in the Route Settings, 'NAT Gateway' will be used to communicate even if floating IP is set for the instance.



## Routing Table 

A routing table is created along with VPC, and is also deleted along with a deletion of VPC. Multiple routing tables can be created within a VPC, and can be deleted explicitly if they are not the default routing table. Subnets must be associated with at least one routing table, and multiple routing tables cannot share an internet gateway.   

When a list of routing table is designated, summary information is displayed on the screen of details, and more routes may be added by using "Routes". 

> [Note]
Adding more routes is available, only when available areas within VPC are designated. Otherwise, message will show failure. 

* Gateways of subnets included to a routing table are automatically added.  

* “Default routing table” cannot be deleted. It shall be deleted along with VPC on its deletion.

* Gateways of subnets and internet cannot be deleted from the list of routers. 

* Disconnecting a routing table from the internet gateway will disconnect the internet. 

A routing table can be created in either a "distributed virtual routing (DVR)" or a "centralized virtual routing (CVR)" method, depending on where it is created.

* Distributed virtual routing (DVR) method

    * A routing table is created for each hypervisor on which an instance in the subnet associated with the routing table is located. Routing tables are distributed evenly across the locations where they are used, providing reliability and high availability.

* Centralized virtual routing (CVR) method

    * One routing table is created at the center, and traffic from instances included in the subnet associated with the routing table is concentrated on one routing table.<br>This method is used when traffic is controlled through a single point, such as a gateway or firewall.

The DVR method is the default method provided by NHN Cloud. It is recommended to use the DVR method except for special circumstances because the method offers advantages of reliability, high availability, and traffic distribution.

It is also possible to change the routing table method. Please note that, when changing the routing table method, external communication and inter-subnet communication will be cut off for about 1 minute until the reconfiguration of routing table is completed.

## Peering 

Peering refers to connecting two different VPCs. In general, VPCs cannot communicate with each other because they are in different network areas, and may be linked via floating IPs but it incurs extra charges depending on the network usage. Therefore, the capability to connect two VPCs is provided, which is called peering.

> [Note] Peering connects two different VPCs. It does not support a connection to another VPC through a VPC. For example, in A <-> B <-> C, A and C cannot be linked.

* IP address areas of two VPCs cannot overlap.<br>
Each IP address area must not be a subset of the other. Otherwise, peering creation will fail.
* Except for the Korea (Pyeongchon) region, communication with subnets not associated with the "default routing table" is not possible.
    * In the Korea (Pyeongchon) region, after creating a peering, separate routes must be set in the routing tables of both peered VPCs to enable communication.
        * Add the route by entering the IP address area of the target VPC in the "Target CIDR" of the route, and selecting the "PEERING" entry with the name of the peering in the "Gateway" list.
        * Communication is possible only with subnets associated with the routing table to which the route has been added.
        * For a routing table that is not the "default routing table", if the routes are added to the routing table, peering communication becomes available on the subnets associated with the routing table.
        * If you specify a VPC without a subnet when creating a peering, the peering is created, but the peering is not actually connected, so you cannot set routes in the routing table. You can set routes after creating at least one subnet in the VPC.


## Co-location gateway

The co-location gateway is a feature used to connect customer network, which is provided as a hybrid service by NHN Cloud. This feature is available only in the Pyeongchon region, Korea. If a hybrid service is used in NHN Cloud, NHN Cloud Zone is provided. It can be used to directly connect to VPC by configuring the co-location gateway.

* A single VPC is connected to a single NHN Cloud Zone one-on-one.
* The fee is charged the moment the VPC is connected.



