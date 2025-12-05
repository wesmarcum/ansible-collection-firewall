Ansible Collection - Firewall
=============================

[![Galaxy Collection][badge-galaxy]][link-galaxy]
[![MIT licensed][badge-license]][link-license]
[![CI][badge-gh-actions]][link-gh-actions]

This collection of roles builds a dedicated open source firewall based on [FreeBSD](https://www.freebsd.org/).  Functionality similar to open source projects PFSense and OPNSense is provided, while maintaining a minimal management layer.

Advantages:

* Build a simple configuration on vanilla FreeBSD.
* Leverage ZFS on FreeBSD with [boot environments](https://wiki.freebsd.org/BootEnvironments).
* No web interface for management, only OpenSSH.
* Configuration management with Ansible and Git.
* Run your configuration on the latest version of FreeBSD and 3rd party packages.

Roles included in this collection:

* [base](./roles/base)
* [inadyn](./roles/inadyn)
* [kea](./roles/kea)
* [monitoring](./roles/monitoring)
* [nsd](./roles/nsd)
* [ntp](./roles/ntp)
* [openvpn_client](./roles/openvpn_client)
* [sanoid](./roles/sanoid)
* [unbound](./roles/unbound)
* [wireguard](./roles/wireguard)

The `base` role is responsible for setting up network interfaces, routing, and the PF firewall.  At a minimum, you should use the following roles in your playbook:

* `wesmarcum.firewall.base`
* `wesmarcum.firewall.unbound` (Recursive DNS resolver)
* `wesmarcum.firewall.kea` (DHCPv4/DHCPv6 server)

See individual role documentation for all configuration parameters.  Sample playbooks and configuration are provided in the `playbooks` and `playbooks/vars` directories.

Installation
------------

Install via Ansible Galaxy:

```
ansible-galaxy collection install wesmarcum.firewall
```

Alternatively, you can include this collection in your playbook's `requirements.yml` file:

```
---
collections:
  - name: wesmarcum.firewall
```

This collection uses the `ipaddr` filter from `ansible.utils`. This requires the Python `netaddr` library on the Ansible controller. This can be installed via pip:

```
pip install netaddr
```

Requirements
------------

It is recommended to use this collection on a dedicated hardware appliance or VM running FreeBSD.  The `base` role modifies `/etc/rc.conf` when needed, but keeps most of the service configuration in separate files located in `/etc/rc.conf.d`.

Before starting, you should have:

* A supported version of FreeBSD.  A fresh install is recommended.
* Python 3.
* Sudo or another method to become root.

Compatibility
-------------

| OS      | Version            |
|---------|--------------------|
| FreeBSD | 14, 15             |

Usage
-----

A simple configuration is provided below.  In order to keep the main playbook minimal, you can read all vars from a file.  This allows you to keep your entire firewall configuration in one yaml file.  Your main playbook can be as simple as the following:

```yaml
---
- name: Set up firewall
  hosts: firewall
  become: false
  vars_files:
    - vars/firewall_simple.yml

  roles:
    - wesmarcum.firewall.base
    - wesmarcum.firewall.unbound
    - wesmarcum.firewall.kea
```

In the file `vars/firewall_simple.yml`, you can then define all variables for the role:

```yaml
---
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

# Define a list of local zones and zone types.
# DNSSEC will be disabled for these zones.
firewall_unbound_local_zones:
  - zone: lan.local.
    type: nodefault

# Define a list of local zones for reverse lookups.
firewall_unbound_local_zones_rev:
  - zone: "10.in-addr.arpa."
  - zone: "d.f.ip6.arpa."

# Disable DHCPv6.  SLAAC can be used instead.
firewall_kea_dhcp6_enable: false

# DHCPv4 configuration.
firewall_kea_dhcp4_config:
  Dhcp4:
    # interfaces to activate for Kea
    interfaces-config:
      interfaces:
        - em1
      # dhcp socket type is 'raw' by default.  Change to 'udp' for relays.
      dhcp-socket-type: raw
  # control socket - uncomment if api is enabled
  # control-socket:
    # socket-type: unix
    # socket-name: "/tmp/kea4-ctrl-socket"
    lease-database:
      type: memfile
      persist: true
      lfc-interval: 3600
      name: /usr/local/var/lib/kea/dhcp4.leases
    expired-leases-processing:
      reclaim-timer-wait-time: 10
      flush-reclaimed-timer-wait-time: 25
      hold-reclaimed-time: 3600
      max-reclaim-leases: 100
      max-reclaim-time: 250
      unwarned-reclaim-cycles: 5
    # global timers
    renew-timer: 10800
    rebind-timer: 21600
    valid-lifetime: 43200
    max-valid-lifetime: 86400
    min-valid-lifetime: 3600
    # Pools
    subnet4:
      - subnet: 10.0.0.0/24
        id: 1000
        interface: em1
        pools:
          - pool: 10.0.0.100 - 10.0.0.200
        option-data:
          - name: routers
            data: 10.0.0.1
          - name: domain-name
            data: lan.local
          - name: domain-name-servers
            data: 10.0.0.1
    loggers:
      - name: kea-dhcp4
        output_options:
          - output: /var/log/kea-dhcp4.log
            pattern: |
              {% raw %}%d{%j %H:%M:%S.%q} %c %m{% endraw %}
            flush: true
            maxsize: 10240000
            maxver: 8
        severity: INFO
        debuglevel: 0
```

#### Uninstalling roles/features

Roles have one or more `enable` variables that can be set to `true` or `false`.  These variables are usually set to `true` by default, which will enable the role and run the tasks.  If set to `false`, tasks intended to uninstall packages and remove configuration files will run.  For example, to uninstall NSD and remove configuration files:

1. Set `firewall_nsd_enable` to `false` in your vars file.
2. Run your playbook to run the uninstall tasks.
3. Remove the `wesmarcum.firewall.nsd` role from your playbook.

License
-------

MIT

Author Information
------------------

<https://github.com/wesmarcum/>

Changelog
---------

1.1.7: Update all tasks/templates to use ansible_facts dictionary.\
1.1.6: Update ip checks for nsd/unbound templates.\
1.1.5: Update paths for kea v2.6.3.\
1.1.4: Add allow list for Unbound.\
1.1.3: Add validation for Kea configuration.\
1.1.2: Add subnet IDs for Kea 2.6.\
1.1.1: Add `trippy` to monitoring role.\
1.1.0: Migrate MTA to `dma` for FreeBSD 14. **Breaking change**: read monitoring documentation and update vars.\
1.0.3: Update cpu-microcode package.\
1.0.2: FreeBSD 13.2 support. Use Wireguard kernel module by default.\
1.0.1: Add support for smartd, microcode updates.\
1.0.0: Initial release.

[badge-license]: https://img.shields.io/badge/license-MIT-green?
[link-license]: https://github.com/wesmarcum/ansible-collection-firewall/blob/main/LICENSE
[badge-gh-actions]: https://github.com/wesmarcum/ansible-collection-firewall/workflows/CI/badge.svg?event=push
[link-gh-actions]: https://github.com/wesmarcum/ansible-collection-firewall/actions?query=workflow%3ACI
[badge-galaxy]: https://img.shields.io/badge/collection-firewall-blue
[link-galaxy]: https://galaxy.ansible.com/wesmarcum/firewall
