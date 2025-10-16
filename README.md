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

## Dependencies

None

## License

MIT

## Author Information

Extracted from homestead infrastructure project.
