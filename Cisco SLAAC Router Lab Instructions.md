![](apnic_logo.png)
# Cisco SLAAC Router Lab 

###Lab Environment
	
* The lab topology has:
	* 1xCSR1000V router
	* 2xLUBUNTU VMs    

### Configure the router:
** When you start the router, it might ask you `Would you like to enter the initial config dialog? [yes/no]`. Please type `no` and proceed.

1. Enable IPv6 routing on the router R1

		ipv6 unicast-routing
		
	
2. Configure the router interface (on which clients connect to) to advertise IPv6 prefix in RA (router advertisement) messages.
	* the A (auto-config) flag is set by default in the RA messages
	* we either do that by advertising a specific IPv6 prefix, or
	* just by configuring an IPv6 address on the interface

   First enable the physical interface (we will use RoAS):

	
		interface FastEthernet0/0
		 description link to IOU-SW
		 no shutdown
 				
 	Then configure the correct sub-interface using either of the following option (preferably the second option)

		interface FastEthernet0/0.100
		 description subinterface for VLAN100
		 encapsulation dot1Q 100
		 ipv6 nd prefix 2406:6400:0:100::/64
OR
			
		interface FastEthernet0/0.100
		 description subinterface for VLAN100
		 encapsulation dot1Q 100
		 ipv6 address 2406:6400:0:100::1/64
		
3. We will also configure Loopback0 to test connectivity

		interface Loopback0
		 ipv6 address 2406:6400::1/128
	

4. Verify your configuration with the following outputs:

		show ipv6 interface ! look at ND stats and different multicast groups joined
		show ipv6 route     ! shows the ipv6 routing table
		show ipv6 neighbors	! the neighbor cache/table
		
5. Enable ICMPv6 ND messages debugging (to see ND messages)
		
		debug ipv6 nd
		
6. Save your configurations
			
		wr	

			
###The Client VM (Lubuntu-1)
		
5. Turn ON (start) both the VMs (Lubuntu_1 and 2). You should be logged in automatically (username and password below)

		username: apnic
		password: root
		
6. Verify that the interface enp0s3 is UP and has computed the IPv6 address using SLAAC
			
		ifconfig
		
		#the address should look something like 2406:6400:0:100:x:x:x:x
		#where the x:x:x:x (64-bit interface ID) is generated randomly (RFC4941)
		#you will see two globally scoped addresses - secured and temporary address (RFC7217 compliant)
		#secured wont change even after reboot, while temporary (outgoing) will
		
			
6. You can also see it from the `Connection Information` dropdown menu

	   ![](Conn_Info.png)

7. In case the interface is not listed or you dont see an IPv6 address, toogle the interface
			
		ifconfig enp0s3 down/up

### Verification:		
* Since you had enabled IPv6 ND debugging on the router, you should see ICMPv6 ND messages being exchanged between the router and the IPv6 clients.

* You should see something similar to below (analyse and understand the messages! Ask your instructors if you dont understand).

		*ICMPv6-ND: Received RS on FastEthernet0/0.100 from FE80::A00:27FF:FE15:F258
		*ICMPv6-ND: Glean FE80::A00:27FF:FE15:F258 on FastEthernet0/0.100
		*ICMPv6-ND: Neighbour FE80::A00:27FF:FE15:F258 on FastEthernet0/0.100 : LLA 0800.2715.f258
		*ICMPv6-ND: INCMP -> STALE: FE80::A00:27FF:FE15:F258
		*ICMPv6-ND: Sending solicited RA on FastEthernet0/0.100
		*ICMPv6-ND: Request to send RA for FE80::C801:6FF:FEE1:0
		*ICMPv6-ND: Setup RA from FE80::C801:6FF:FEE1:0 to FF02::1 on FastEthernet0/0.100
		*ICMPv6-ND:  MTU = 1500
		*ICMPv6-ND:     prefix = 2406:6400:0:100::/64 onlink autoconfig
		*ICMPv6-ND: 	     2592000/604800 (valid/preferred)
		
* Ping each other and also ping the router Loopback0 from the client machines.
		
		ping6 2406:6400::1





