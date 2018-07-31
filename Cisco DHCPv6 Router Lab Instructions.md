![](apnic_logo.png)
# LAB2: Stateless DHCPv6

###Lab Environment
Please continue from the previous lab (SLAAC)
	
* The lab topology still has:
	* 1x7206VXR router
	* 1xCisco IOU switch
	* 2xUbuntu VMs
	
      

### Configure the router:

1. Configure the DHCPv6 server

		ipv6 dhcp pool STATELESS-DHCPv6
		 dns-server 2406:6400::1		!this is the loopback0
		 domain-name nog.bt
		
	
2. Bind the DHCPv6 pool to the interface towards to the client:
	* clients will use the IPv6 prefix in the RA to compute their address using SLAAC, which means the DHCPv6 server will not track or maintain IPv6 addresses used by clients (hence, stateless).
	* the O (other-config) flag in the RA tells the clients to obtain other information (dns, domain, etc) from the DHCPv6 server
	* the A (auto-config) flag is still set (default) in the RAs to the clients
	
			interface FastEthernet0/0.100
			 ipv6 dhcp server STATELESS-DHCPv6
			 ipv6 nd other-config-flag 
			

4. Verify your configuration with the following outputs:

		show ipv6 interface 
		!look at ND stats and different multicast groups joined (anything different?)
		
		show ipv6 route     
		!shows the ipv6 routing table
		
		show ipv6 neighbors
		!list the neigbors
		
5. To see the ICMPv6 ND and DHCPv6 messages
		
		debug ipv6 nd
		debug ipv6 dhcp
	
	* you could also use packet capture if you have wireshark on your host machine, as shown below:
	
			right-click on any link on GNS3 topology, and click "start capture"
		
6. Save your configurations
			
		wr	

### Configure the switch:
1. The switch configuration is same as before (no changes necessary)

			
###The Client VMs (Ubuntu)
		
5. Toogle the interface enp0s3 

		ifconfig enp0s3 down/up		!need sudo
		
6. Verify that the interface enp0s3 is UP and the IPv6 address is still computed using SLAAC (A-flag in the RA)
			
		ifconfig
		
		#the address should look something like 2406:6400:0:100:x:x:x:x
		#where the x:x:x:x (64-bit interface ID) is generated randomly (RFC4941)
		#you will see two globally scoped addresses - secured and temporary address (RFC7217 compliant)
		#secured wont change even after reboot, while temporary (outgoing) will		
6. Further, verify that "Other" stateful information (dns server in this case) are obtained from the DHCPv6 server: you can see it from the `Connection Information` dropdown menu

	   ![](o-flag.png)
	   

### Verification:		
* Since you had enabled IPv6 ND and DHCPv6 debugging on the router, you should see both ICMPv6 ND and DHCPv6 messages being exchanged between the router and the IPv6 clients.

* You should see something like below on your router (analyse and understand the messages! Ask your instructors if you dont understand).

		*ICMPv6-ND: Received RS on FastEthernet0/0.100 from FE80::EA73:6EA6:353C:586
		*ICMPv6-ND: Sending solicited RA on FastEthernet0/0.100
		*CMPv6-ND: Request to send RA for FE80::C802:7FF:FE9A:0
		*ICMPv6-ND: Setup RA from FE80::C802:7FF:FE9A:0 to FF02::1 on FastEthernet0/0.100
		*ICMPv6-ND: Setup RA common:Other stateful configuration
		*ICMPv6-ND:  MTU = 1500
		*ICMPv6-ND:     prefix = 2406:6400:0:100::/64 onlink autoconfig
		*ICMPv6-ND: 	     2592000/604800 (valid/preferred)
		*IPv6 DHCP: Received INFORMATION-REQUEST from FE80::EA73:6EA6:353C:586 on FastEthernet0/0.100
		*IPv6 DHCP: Using interface pool STATELESS-DHCPv6
		*IPv6 DHCP_AAA: Retrieved subblock; It has AAA DNS_SERVERS=0
		*IPv6 DHCP: Source Address from SAS FE80::C802:7FF:FE9A:0
		*IPv6 DHCP: Sending REPLY to FE80::EA73:6EA6:353C:586 on FastEthernet0/0.100
		*ICMPv6-ND: Created ND Entry Chunk pool
		*ICMPv6-ND: DELETE -> INCMP: FE80::EA73:6EA6:353C:586
		*ICMPv6-ND: Sending NS for FE80::EA73:6EA6:353C:586 on FastEthernet0/0.100
		*ICMPv6-ND: Resolving next hop FE80::EA73:6EA6:353C:586 on interface FastEthernet0/0.100
		*ICMPv6-ND: Received NA for FE80::EA73:6EA6:353C:586 on FastEthernet0/0.100 from FE80::EA73:6EA6:353C:586
		*ICMPv6-ND: Neighbour FE80::EA73:6EA6:353C:586 on FastEthernet0/0.100 : LLA 0800.2775.0fc2
		*ICMPv6-ND: INCMP -> REACH: FE80::EA73:6EA6:353C:586
		*ICMPv6-ND: Received NS for FE80::C802:7FF:FE9A:0 on FastEthernet0/0.100 from FE80::EA73:6EA6:353C:586
		*ICMPv6-ND: Sending NA for FE80::C802:7FF:FE9A:0 on FastEthernet0/0.100

* Ping each other and also ping the router loopback0 (simulating the DNS server in this case).