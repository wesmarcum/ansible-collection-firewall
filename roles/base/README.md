Firewall - Base
===============

The `base` role is required for all other roles in the `wesmarcum.firewall` collection.  This role sets interface parameters using `firewall_interfaces`, pf options and rule sets (with optional Geo IP blocking), DHCPv4 / DHCPv6 clients, and IPv6 router advertisements using `radvd`.

Features:
 - Interface configuration.
 - Set default and static routes.
 - Installation of common packages.
 - PF configuration.
 - Geo IP blocking (optional).
 - IPv6 router advertisements with `radvd`.
 - DHCPv6 client using `dhcp6c`.

Installation
------------

See documentation for the collection `wesmarcum.firewall`.

Requirements
------------

No external roles/collections are required.

Before using this role, you should be familiar with the following:
 - The `firewall_interfaces` variable/dictionary.
 - The default pf rules or using a custom template.

PF rules are applied using the template `templates/etc/pf/pf.conf_default.j2` by default.  This template provides a quick method to get a rule set in place.  If you use this template, your external interface must be named "outside" and your internal interface must be named "inside".

If you wish to use your own template for the PF rule set, provide a template in your playbooks directory with the `hostname` of the firewall in place of `default`.  For example, if the hostname is `myfirewall`, your rule set template should be named `templates/etc/pf/pf.conf_myfirewall.j2`.  The base role will then use your custom template instead of the default.

Role Variables
--------------

#### General:

| Variable Name                     | Default Value         | Description                                       |
|-----------------------------------|-----------------------|---------------------------------------------------|
| firewall_zfs_enable               | true                  | Enable zfs in rc.conf.                            |
| firewall_powerd_enable            | true                  | Enable powerd for cpu frequency scaling.          |
| firewall_powerd_flags             | empty                 | Flags for powerd at startup.                      |
| firewall_smartd_enable            | false                 | Enable smartd for disk monitoring.                |
| firewall_smartd_email             | root                  | User or email address to send alerts from smartd. |
| firewall_microcode_updates_enable | false                 | Update microcode for intel/amd CPUs.              |
| firewall_common_packages          | see defaults/main.yml | Packages to install for base.                     |

#### Network / Routing:

| Variable Name                | Default Value | Description                                                                               |
|------------------------------|---------------|-------------------------------------------------------------------------------------------|
| firewall_hostname            | "fw.local"    | The hostname for the firewall.                                                            |
| firewall_interfaces          | empty         | A dictionary containing the interface configuration.  See interface table for parameters. |
| firewall_ipv4_gateway_enable | YES           | Enable routing for IPv4.                                                                  |
| firewall_ipv6_gateway_enable | YES           | Enable routing for IPv6.                                                                  |
| firewall_ipv4_default_route  | empty         | Manually set a default route for IPv4.                                                    |
| firewall_ipv6_default_route  | empty         | Manually set a default route for IPv6.                                                    |
| firewall_static_routes       | empty         | A list of dictionary values to define static routes.  See description below.              |

The `firewall_interfaces` variable is a nested dictionary that defines all interface configuration.  This **must** be defined before using the base role.  The first level of the dictionary should match the interface name on FreeBSD (e.g. `em0`).  The second level defines all parameters of the interface.  List of parameters:

| Interface Parameter | Examples                         | Description                                                                         |
|---------------------|----------------------------------|-------------------------------------------------------------------------------------|
| name                | "outside" or "inside"            | A short, descriptive name for the interface.  One word in lowercase is recommended. |
| external            | true or false                    | Boolean value to determine if the interface is external (untrusted).                |
| ipv4_address        | "dhcp" or "10.1.1.1/24"          | IPv4 address in CIDR notation to assign.  Set to "dhcp" for DHCPv4.                 |
| ipv6_address        | "dhcp" or "fd01:0002:0003::1/64" | IPv6 address in CIDR notation to assign.  Set to "dhcp" for DHCPv6.                 |
| ipv6_cpe_wanif      | true or false                    | Enable 'ipv6_cpe_wanif' for FreeBSD.  This accepts router advertisements.           |
| ipv6_send_ia_pd     | 0 or [0, 1]                      | Request IPv6 prefix delegation for these IDs.  Can be a single value or list.       |
| ipv6_pd             | 0                                | Assign a prefix delegation to an interface.  Single integer value.                  |
| ipv6_pd_sla_id      | 0                                | ID of the SLA (site level aggregator).  Combined with PD to form the subnet.        |
| ipv6_pd_sla_len     | 0                                | Length of the SLA ID in bits.                                                       |
| domain              | "lan.local"                      | DNS domain for the interface.  Used for `radvd`.                                    |
| mtu                 | 1500                             | MTU value for the interface.                                                        |
| state               | up or down                       | Define administrative state for the interface.                                      |
| vlans               | [2, 3, 4]                        | List of vlans to create for the physical interface.                                 |

For example, to define `em0` as an external interface, and `em1` as an internal interface:
```yaml
firewall_interfaces:
  em0:
    name: "outside"
    external: true
    ipv4_address: "dhcp"
    ipv6_address: "dhcp"
    ipv6_cpe_wanif: true
  em1:
    name: "inside"
    ipv4_address: "10.0.0.1/24"
    ipv6_address: "fd00:0001:0002::1/64"
    domain: "lan.local"
    mtu: 1500
```

The example above will set the "outside" interface to dhcp for both IPv4 and IPv6.  The "inside" interface will be assigned the static addresses defined by the `ipv4_address` and `ipv6_address` variables.  Note that either `ipv4_address` or `ipv6_address` can be omitted if they are not needed.

VLANs must be configured as a list using the `vlans` property on the physical interface.  The state of the physical interface must also be set to `up`.  The VLAN sub-interfaces can then be configured by using `<interface>.<vlan id>` notation (e.g. `em0.2`):
```yaml
firewall_interfaces:
  em0:
    state: up
    vlans:
      - 2
      - 3
  em0.2:
    name: "inside"
    ipv4_address: "10.0.0.1/24"
    ipv6_address: "fd00:0001:0002::1/64"
    domain: "lan.local"
    mtu: 1500
  em0.3:
    name: "guest"
    ipv4_address: "10.0.1.1/24"
    ipv6_address: "fd00:0001:0003::1/64"
    domain: "guest.local"
```

Static routes may be defined with the `firewall_static_routes` variable.  This variable is a list of static routes, with each route defined as a dictionary.  Note that all values in the table below are mandatory when defining routes.

| Parameter | Examples       | Description                            |
|-----------|----------------|----------------------------------------|
| name      | net1 or net2   | The name of the static route.          |
| network   | 192.168.0.0/24 | The destination network for the route. |
| next_hop  | 192.168.1.1    | The IP address of the next hop.        |

Example static routes:
```yaml
firewall_static_routes:
  - name: net1
    network: 192.168.0.0/24
    next_hop: 192.168.1.1
  - name: net2
    network: 192.168.1.0/24
    next_hop: 192.168.1.1
```

#### PF:

| Variable Name             | Default Value     | Description                                                                        |
|---------------------------|-------------------|------------------------------------------------------------------------------------|
| firewall_pf_rules         | "/etc/pf/pf.conf" | Location for pf rules/configuration.                                               |
| firewall_pf_logfile       | "/var/log/pflog"  | Location for the pf logfile.                                                       |
| firewall_pf_tables        | empty             | List of tables to copy for pf.  Tables should be located in "files/etc/pf/tables". |
| firewall_pf_table_entries | 4000000           | Max number of addresses that can be stored in tables.                              |
| firewall_pf_limit_states  | 800000            | Max number of entries in the memory pool used by the state table entries.          |
| firewall_pf_src_nodes     | 800000            | Max entries for tracking source IP addresses.                                      |

The `firewall_pf_tables` variable can be set to a list of table names to copy to the remote system.  Tables must be located in `files/etc/pf/tables` on the host running the playbook.

#### RADVD:

| Variable Name               | Default Value | Description                                    |
|-----------------------------|---------------|------------------------------------------------|
| firewall_radvd_enable       | true          | Enable `radvd` for IPv6 route advertisements.  |
| firewall_radvd_uid          | 991           | User ID for `radvd`.                           |
| firewall_radvd_gid          | 991           | Group ID for `radvd`.                          |
| firewall_radvd_min_interval | 5             | Time in seconds between sending RAs (minimum). |
| firewall_radvd_max_interval | 20            | Time in seconds between sending RAs (maximum). |

#### IPv6 Prefix Delegation:

| Variable Name           | Default Value   | Description                                                          |
|-------------------------|-----------------|----------------------------------------------------------------------|
| firewall_ipv6_pd_enable | false           | Enable or disable IPv6 prefix delegation support.                    |
| firewall_ipv6_pd_list   | see table below | Global list/dictionary of IPv6 prefix delegation IDs and attributes. |

IPv6 prefix delegation can be enabled globally by setting `firewall_ipv6_pd_enable` to `true`.  Prefix delegation can use DHCPv6 to request a block of external IP addresses from your ISP.  The block can then be assigned to internal interfaces.  Prefix delegation parameters can be specified globally with the `firewall_ipv6_pd_list` variable.  This variable is a list of one or more prefix delegations with the following dictionary values:

| Parameter     | Examples          | Description                                                    |
|---------------|-------------------|----------------------------------------------------------------|
| **id**        | 0                 | ID (integer) for the prefix delegation.                        |
| prefix_length | ::/64             | Requested prefix length for the delegation.  Default is ::/64. |
| lifetime      | 86400 or infinity | PD lifetime in seconds or infinity.  Default is infinity.      |

Note: The prefix length defaults to `::/64`.  _Most_ ISPs will delegate blocks of `::/64`, `::/60`, or `::/56`.  If you need a prefix for multiple subnets, and your ISP only delegates blocks of `::/64`, you will need to use multiple prefix delegations - one for each internal interface.

Once global PD settings are defined, you just need to set the corresponding options in the `firewall_interfaces` dictionary.  See the main `firewall_interfaces` table for details.

Example with one PD:
```yaml
firewall_ipv6_pd_enable: true
firewall_ipv6_pd_list:
  - id: 0
    prefix_length: ::/60
    lifetime: infinity

firewall_interfaces:
  em0:
    name: "outside"
    external: true
    ipv4_address: "dhcp"
    ipv6_address: "dhcp"
    ipv6_cpe_wanif: true
    ipv6_send_ia_pd:        # Request PD with ID 0 on the external interface.
      - 0
  em1:
    name: "inside"
    ipv4_address: "10.0.0.1/24"
    ipv6_address: "fd00:0001:0002::1/64"
    ipv6_pd: 0              # Assign a subnet from PD ID 0 to this interface.
    ipv6_pd_sla_id: 0       # Define ID of the SLA.  This integer is combined with the PD to form subnet ID.
    ipv6_pd_sla_len: 4      # Length of the SLA ID in bits.
    domain: "lan.local"
    mtu: 1500
  em2:
    name: "guest"
    ipv4_address: "10.0.1.1/24"
    ipv6_address: "fd00:0001:0003::1/64"
    ipv6_pd: 0
    ipv6_pd_sla_id: 1
    ipv6_pd_sla_len: 4
    domain: "guest.local"
```
_Note:_ The `ipv6_pd_sla_id` and `ipv6_pd_sla_len` variables can be omitted, but they will default to 0.

Example with multiple PDs:
```yaml
firewall_ipv6_pd_enable: true
firewall_ipv6_pd_list:
  - id: 0
    prefix_length: ::/64
    lifetime: infinity
  - id: 1
    prefix_length: ::/64
    lifetime: infinity

firewall_interfaces:
  em0:
    name: "outside"
    external: true
    ipv4_address: "dhcp"
    ipv6_address: "dhcp"
    ipv6_cpe_wanif: true
    ipv6_send_ia_pd:        # Request PD with IDs 0 and 1 on the external interface.
      - 0
      - 1
  em1:
    name: "inside"
    ipv4_address: "10.0.0.1/24"
    ipv6_address: "fd00:0001:0002::1/64"
    ipv6_pd: 0              # Assign a subnet from PD ID 0 to this interface.
    ipv6_pd_sla_id: 0       # Define ID of the SLA.  This integer is combined with the PD to form subnet ID.
    ipv6_pd_sla_len: 0      # Length of the SLA ID in bits.
    domain: "lan.local"
    mtu: 1500
  em2:
    name: "guest"
    ipv4_address: "10.0.1.1/24"
    ipv6_address: "fd00:0001:0003::1/64"
    ipv6_pd: 1
    ipv6_pd_sla_id: 0
    ipv6_pd_sla_len: 0
    domain: "guest.local"
```

#### Geo IP blocking:

| Variable Name                | Default Value                      | Description                                                                                                  |
|------------------------------|------------------------------------|--------------------------------------------------------------------------------------------------------------|
| firewall_geoip_enable        | false                              | Enable geoip blocking.  This auto-generates tables that can be used in pf rules.                             |
| firewall_geoip_country_codes | empty                              | List of [ISO 3166 country codes](https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes) for blocking. |
| firewall_geoip_include_ipv6  | true                               | Include IPv6 prefixes and generate the corresponding table.                                                  |
| firewall_geoip_ipv4_file     | "/etc/pf/tables/geoip_blocks_ipv4" | Location for the IPv4 block table.                                                                           |
| firewall_geoip_ipv6_file     | "/etc/pf/tables/geoip_blocks_ipv6" | Location for the IPv6 block table.                                                                           |

If `firewall_geoip_enable` is set to `true`, tables will be auto-generated for use in PF rules.  The `firewall_geoip_country_codes` list should be populated with [ISO 3166 country codes](https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes) of the countries you would like to include in the PF tables.  IPv4 and IPv6 ranges are downloaded with the FreeBSD [ipdbtools](https://www.freshports.org/sysutils/ipdbtools/) package and updated weekly.

The `firewall_geoip_ipv4_file` and `firewall_geoip_ipv6_file` variables determine where the generated tables are stored for PF.  You can then set up tables for PF and use them in rule sets like the following:

```
table <geoblock_ipv4> persist file "{{ firewall_geoip_ipv4_file }}"
table <geoblock_ipv6> persist file "{{ firewall_geoip_ipv6_file }}"

block in log quick on $outside inet from <geoblock_ipv4> to any label "block conns from geoblocked countries"
block in log quick on $outside inet6 from <geoblock_ipv6> to any label "block conns from geoblocked countries"
```

Note that this configuration is included in the default rule set `templates/etc/pf/pf.conf_default.j2`.

Example Playbook
----------------

In order to keep the main playbook minimal, you can read all vars from a file.  This allows you to keep your entire firewall configuration in one yaml file.  The playbook can be as simple as the following:

```yaml
---
- name: Set up firewall
  hosts: firewall
  become: false
  vars_files:
    - vars/firewall.yml

  roles:
    - wesmarcum.firewall.base
```

In the file `vars/firewall.yml`, you can then define all variables for the role:
```yaml
---
firewall_hostname: "firewall.local"
firewall_pf_tables:
  - mytable

firewall_geoip_enable: true
firewall_geoip_country_codes:
  - RU
  - CN

firewall_interfaces:
  em0:
    name: "outside"
    external: true
    ipv4_address: "dhcp"
    ipv6_address: "dhcp"
    ipv6_cpe_wanif: true
  em1:
    state: up
    vlans:
      - 2
      - 3
  em1.2:
    name: "inside"
    ipv4_address: "10.0.0.1/24"
    ipv6_address: "fd00:0001:0002::1/64"
    domain: "lan.local"
    mtu: 1500
  em1.3:
    name: "guest"
    ipv4_address: "10.0.1.1/24"
    ipv6_address: "fd00:0001:0003::1/64"
    domain: "guest.local"
```

License
-------

MIT

Author Information
------------------

https://github.com/wesmarcum/
