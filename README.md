# Ansible Role: systemd-networkd

Configure systemd-networkd for network interface management on Linux systems.

## Requirements

- systemd-based Linux distribution
- Ansible 2.9 or higher

## Role Variables

### systemd_networkd_interfaces

A list of network interfaces to configure. Each interface should define:

- `interface_name`: Name of the interface
- `interface_type`: Type of interface (bond, bridge, dhcp, bond-member, bridge-member)
- `config_type`: Configuration file type (netdev, network, link)
- `order`: Numeric ordering for systemd-networkd (e.g., 10, 20, 30)
- Additional type-specific parameters (e.g., `mac_address` for bonds)

Default: `[]` (empty list)

## Example Playbook

```yaml
- hosts: servers
  roles:
    - role: systemd_networkd
      systemd_networkd_interfaces:
        - interface_name: bond0
          interface_type: bond
          config_type: netdev
          order: 10
          mac_address: "52:54:00:12:34:56"

        - interface_name: eno1
          interface_type: bond-member
          config_type: network
          order: 11
          bond_name: bond0

        - interface_name: bond0
          interface_type: dhcp
          config_type: network
          order: 20
```

## Supported Interface Types

- **bond**: Create bonded interfaces (active-backup mode)
- **bridge**: Create bridge interfaces
- **dhcp**: Configure DHCP on interfaces
- **vlan**: Create tagged VLAN interfaces (802.1Q)
- **bond-member**: Add interface to bond
- **bridge-member**: Add interface to bridge

## Templates

The role uses Jinja2 templates for different interface types:

- `bond.netdev.j2`: Bond device configuration
- `bond.link.j2`: Bond link configuration
- `bond-member.network.j2`: Bond member configuration
- `bridge.netdev.j2`: Bridge device configuration
- `bridge.network.j2`: Bridge network configuration
- `bridge-member.network.j2`: Bridge member configuration
- `dhcp.network.j2`: DHCP network configuration
- `vlan.netdev.j2`: VLAN device configuration
- `vlan.network.j2`: VLAN network configuration

### Bond Configuration

Bonds are configured with the following settings:
- Mode: active-backup
- Primary Reselect Policy: always
- MII Monitor: 1s
- Up Delay: 2s
- Down Delay: 5s

### DHCP Configuration

DHCP-configured interfaces will:
- Send the hostname from Ansible inventory
- Request both IPv4 and IPv6 addresses

### VLAN Configuration

VLAN interfaces support 802.1Q tagged VLANs and can be:
- Attached to bridge interfaces
- Configured with static IP addressing
- Used as standalone tagged interfaces

Example configuration for VLANs on bridges:

```yaml
systemd_networkd_interfaces:
  # Configure parent interface with VLAN declarations
  - interface_name: eth0
    interface_type: dhcp
    config_type: network
    order: 10
    vlans:
      - vlan100
      - vlan200

  # Create bridge for VLAN 100
  - interface_name: br100
    interface_type: bridge
    config_type: netdev
    order: 20

  # Create VLAN 100 device
  - interface_name: vlan100
    interface_type: vlan
    config_type: netdev
    order: 25
    vlan_id: 100

  # Attach VLAN 100 to bridge
  - interface_name: vlan100
    interface_type: vlan
    config_type: network
    order: 30
    parent_interface: eth0
    bridge_name: br100

  # Configure bridge with IP
  - interface_name: br100
    interface_type: bridge
    config_type: network
    order: 35
    ipv4_address: 192.168.100.1
    ipv4_prefix: 24
    ipv4_forwarding: true

  # Create bridge for VLAN 200
  - interface_name: br200
    interface_type: bridge
    config_type: netdev
    order: 40

  # Create VLAN 200 device
  - interface_name: vlan200
    interface_type: vlan
    config_type: netdev
    order: 45
    vlan_id: 200

  # Attach VLAN 200 to bridge
  - interface_name: vlan200
    interface_type: vlan
    config_type: network
    order: 50
    parent_interface: eth0
    bridge_name: br200

  # Configure bridge with IP
  - interface_name: br200
    interface_type: bridge
    config_type: network
    order: 55
    ipv4_address: 192.168.200.1
    ipv4_prefix: 24
```

## Dependencies

None

## License

MIT

## Author Information

Extracted from homestead infrastructure project.
