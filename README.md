# ansible-interfaces

This roles aims to configure Debian network interfaces.

The supported of interfaces are these:
-  Bonded interfaces
-  Bridge interfaces
-  Ethernet interfaces
-  Network routes
-  VLAN tagged interfaces

## Requirements

This role requires Ansible >=2.6. Platform requirements are listed
in the metadata file.

## Role Variables

The variables that can be passed to this role and a brief description about
them are as follows:

| Variable                    | Required | Default | Comments                           |
|-----------------------------|----------|---------|------------------------------------|
| `network_ether_interfaces`  | No       | `[]`    | list of Ethernet interfaces to add |
| `network_bridge_interfaces` | No       | `[]`    | list of bridge interfaces to add   |
| `network_bond_interfaces`   | No       | `[]`    | list of bonded interfaces to add   |
| `network_vlan_interfaces`   | No       | `[]`    | list of VLAN interfaces to add     |

Note: The values for the list are listed in the examples below.

## Examples

Debian network configurations can optionally use CIDR notation for IPv4 addresses instead of specifying the address and subnet mask separately.

It is required to use CIDR notation for IPv6 addresses on Debian.

IPv4 example with CIDR notation:
```yaml
      cidr: 192.168.10.18/24
      # OPTIONAL: specify a gateway for that network, or auto for network+1
      gateway: auto
```

IPv4 example:
```yaml
      address: 192.168.10.18
      netmask: 255.255.255.0
      network: 192.168.10.0
      broadcast: 192.168.10.255
      gateway: 192.168.10.1
```

If you want to use a different MAC address for your interface, you can simply add it:
```yaml
      hwaddress: aa:bb:cc:dd:ee:ff
```

On some rare occasion it might be good to set whatever option you like. Therefore it
is possible to use:
```yaml
      options:
          - "up /execute/my/command"
          - "down /execute/my/other/command"
```
and the IPv6 version
```yaml
      ipv6_options:
          - "up /execute/my/command"
          - "down /execute/my/other/command"
```

1) Configure eth1 and eth2 on a host with a static IP and a DHCP IP. Also
define static routes and a gateway.

```yaml
- hosts: myhost
  roles:
    - role: network
      network_ether_interfaces:
       - device: eth1
         bootproto: static
         cidr: 192.168.10.18/24
         gateway: auto
         route:
          - network: 192.168.200.0
            netmask: 255.255.255.0
            gateway: 192.168.10.1
          - network: 192.168.100.0
            netmask: 255.255.255.0
            gateway: 192.168.10.1
       - device: eth2
         bootproto: dhcp
```
Note: it is not required to add routes, default route will be added automatically.

2) Configure a bridge interface with multiple NICs added to the bridge.

```yaml
- hosts: myhost
  roles:
    - role: network
      network_bridge_interfaces:
       -  device: br1
          type: bridge
          cidr: 192.168.10.10/24
          bridge_ports: [eth1, eth2]
          # Optional values
          bridge_ageing: 300
          bridge_bridgeprio: 32768
          bridge_fd: 15
          bridge_gcint: 4
          bridge_hello: 2
          bridge_maxage: 20
          bridge_maxwait: 0
          bridge_pathcost: "eth1 100"
          bridge_portprio: "eth1 128"
          bridge_stp: "on"
          bridge_waitport: "5 eth1 eth2"
```
Note: Routes can also be added for this interface in the same way routes are
added for ethernet interfaces.

3) Configure a bond interface with an "active-backup" slave configuration.

```yaml
- hosts: myhost
  roles:
    - role: network
      network_bond_interfaces:
        - device: bond0
          address: 192.168.10.128
          netmask: 255.255.255.0
          bond_mode: active-backup
          bond_slaves: [eth1, eth2]
          # Optional values
          bond_miimon: 100
          bond_lacp_rate: slow
          bond_xmit_hash_policy: layer3+4
```

4) Configure a bonded interface with "802.3ad" as the bonding mode and IP
address obtained via DHCP.

```yaml
- hosts: myhost
  roles:
    - role: network
      network_bond_interfaces:
        - device: bond0
          bootproto: dhcp
          bond_mode: 802.3ad
          bond_miimon: 100
          bond_slaves: [eth1, eth2]
          bond_ad_select: 2
```

5) Configure a VLAN interface with the VLAN ID 2 for an Ethernet interface

```yaml
- hosts: myhost
  roles:
    - role: network
      network_ether_interfaces:
       - device: eth1
         bootproto: static
         cidr: 192.168.10.18/24
         gateway: auto
      network_vlan_interfaces:
       - device: eth1.2
         bootproto: static
         cidr: 192.168.20.18/24
```

8) You can add IPv6 static IP configuration on Ethernet, Bond or Bridge interfaces

```yaml
ipv6_address: "aaaa:bbbb:cccc:dddd:dead:beef::1/64"
ipv6_gateway: "aaaa:bbbb:cccc:dddd::1"
```

## Dependencies

- `python-netaddr`

## License

BSD

## Authors Information

This role is a fork, aiming to manage only Debian systems with a refactored code.

This project was originally created by [Benno Joy](https://github.com/bennojoy/network_interface).

Debian upgrades by:

* Martin Verges (croit, GmbH)
* Eric Anderson (Avi Networks, Inc.)

RedHat upgrades by:

* Eric Anderson (Avi Networks, Inc.)
* Luke Short (Red Hat, Inc.)
* Wei Tie, (Cisco Systems, Inc.)
