# Ansible Connectivity

## Bootstrap

For intial day 0 boot strapping of the network, I would ideally use something like autoinstall, where network devices get a mgmt IP address from a DHCP server, and download an initial configuration from a TFTP server, allowing ansible ssh access to make further configurations. For the Cisco vIOS images this is not available, so I simulated this by adding initial configuration files in EVE. Below shows the router initial config and the initial config from one of the switches:

```
hostname EDGE1
!  
ip domain-name my.lab
username admin privilege 15 secret ansible!
ip ssh version 2
!
interface GigabitEthernet0/0
 description MGMT-BOOT
 ip address 10.100.99.11 255.255.255.0
 no shutdown
!
line vty 0 4
 login local
 transport input ssh
!
end
```
```
hostname DSW1
!
ip domain-name my.lab
username admin privilege 15 secret ansible!
ip ssh version 2
!
vlan 99
!
int vlan99
 ip address 10.100.99.12 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/0
 description MGMT-BOOT
 switchport mode access
 switchport access vlan 99
 no shutdown
!
line vty 0 4
 login local
 transport input ssh
!
end
```

I then ran the "discover_devices.yml" playbook which gathered some basic facts about each device:

![alt text](image.png)

Now we have connectivity via ssh to the devices from ansible, and can begin with network configuration.