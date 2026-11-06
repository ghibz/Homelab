Network	Bridge	Subnet	Gateway	Notes
default	virbr0	192.168.122.0/24	192.168.122.1	NAT, OPNsense WAN
vlan10-users	virbr1	10.10.10.0/24	10.10.10.1	DHCP .100-.200
vlan20-servers	virbr2	10.10.20.0/24	10.10.20.1	static only
vlan30-mgmt	virbr3	10.10.30.0/24	10.10.30.1	Mint host = .2
vlan40-dmz	virbr4	10.10.40.0/24	10.10.40.1	web server later
