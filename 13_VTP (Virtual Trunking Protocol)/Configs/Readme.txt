# VTP(Virtual Trunking Protocol) Configuration

Rules : All the links must be trunk links.
	Domain name must match on all devices.
	Password must match.

# SW-1 (Server) Configuration

en
conf t 
host SW-1

int fa0/1
switchport mode trunk
exit

vtp domain CCNA
vtp mode server
vtp ver 2
vtp password CCNA@8835

end
wr


# SW-2 (Client) Configuration

en
conf t 
host SW-2

int fa0/2e
switchport mode trunk

exit

vtp domain CCNA
vtp mode client
vtp ver 2
vtp password CCNA@8835

end 
wr

# SW-3 (Transparent) Configuration

en
conf t 
host SW-3

int fa0/2
switchport mode trunk
exit

vtp domain CCNA
vtp mode transparent
vtp ver 2
vtp password CCNA@8835

end
wr


# Testing

:- Creating Vlan on SW-1 (Server).

en
conf t
vlan 10
name HR
vlan 20
name MR

* Same Configuration will be advertised to client and transparent switch
  via trunk port.

* Verify using show vtp status command
