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
## Initial Connectivity and Backup
Next, I wanted to establish initial connectivity and take a backup of the running config for each device. For this I ran my first playbook, shown below:

```
---
- name: Validate access and backup Cisco IOS devices
  hosts: ios_routers:ios_switches
  gather_facts: false

  tasks:
    - name: Collect basic device facts
      cisco.ios.ios_facts:
        gather_subset:
          - min

    - name: Display device summary
      ansible.builtin.debug:
        msg:
          - "Inventory name: {{ inventory_hostname }}"
          - "Management address: {{ ansible_host }}"
          - "Device hostname: {{ ansible_net_hostname }}"
          - "IOS version: {{ ansible_net_version }}"
          - "Model: {{ ansible_net_model }}"
    - name: Back up the running configuration
      cisco.ios.ios_config:
        backup: true
        backup_options:
          dir_path: "{{ backup_directory }}"
        save_when: never
      register: backup_result
      
    - name: Display backup location
      ansible.builtin.debug:
        var: backup_result.backup_path
```
This goes through each host and gathers some basic facts including the hostname, IOS version, management IP etc. It backs up the running config of each host and saves them to a backups directory. Below is an example of the output from the playbook, proving SSH connectivity via ansible:

```
ok: [DSW1] => {
    "msg": [
        "Inventory name: DSW1",
        "Management address: 10.100.99.12",
        "Device hostname: DSW1",
        "IOS version: 15.2(20200924:215240)",
        "Model: IOSv"
    ]
}
ok: [ASW1] => {
    "msg": [
        "Inventory name: ASW1",
        "Management address: 10.100.99.14",
        "Device hostname: ASW1",
        "IOS version: 15.2(20200924:215240)",
        "Model: IOSv"
    ]
}
ok: [EDGE1] => {
    "msg": [
        "Inventory name: EDGE1",
        "Management address: 10.100.99.11",
        "Device hostname: EDGE1",
        "IOS version: 15.9(3)M6",
        "Model: IOSv"
    ]
}
ok: [DSW2] => {
    "msg": [
        "Inventory name: DSW2",
        "Management address: 10.100.99.13",
        "Device hostname: DSW2",
        "IOS version: 15.2(20200924:215240)",
        "Model: IOSv"
    ]
}
ok: [ASW2] => {
    "msg": [
        "Inventory name: ASW2",
        "Management address: 10.100.99.15",
        "Device hostname: ASW2",
        "IOS version: 15.2(20200924:215240)",
        "Model: IOSv"
    ]
}
```

Now we have connectivity via ssh to the devices from ansible, and can begin with network configuration.