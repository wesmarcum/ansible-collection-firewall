Firewall - NTP
==============

The `ntp` role activates and configures the NTP server.  The role automatically configures the North America NTP pool, so minimal configuration should be required.

Installation
------------

See documentation for the collection `wesmarcum.firewall`.

Requirements
------------

The role `wesmarcum.firewall.base` is required.  This role reads the `firewall.interfaces` variable and automatically generates configuration.

The variable `firewall_ntp_interfaces` should be defined.  This provides a list of interfaces to enable for serving NTP.  Normally, this should be a list of trusted internal interfaces.  By default, `firewall_ntp_pool` will point to the North America NTP pools, but this can be overridden.  Specific NTP servers can optionally be defined by providing `firewall_ntp_servers`.  The `firewall_ntp_permit_networks` list can define an optional list of networks.  This can be used to permit VPN or remote networks.

Role Variables
--------------

#### General:

| Variable Name                 | Default Value                        | Description                                                                           |
|-------------------------------|--------------------------------------|---------------------------------------------------------------------------------------|
| firewall_ntp_enable           | true                                 | Enable or disable the NTP server.                                                     |
| firewall_ntp_conf             | /etc/ntp.conf                        | Location of the NTP configuration file.                                               |
| firewall_ntp_db_path          | /var/db/ntp                          | Location of the NTP DB.                                                               |
| firewall_ntp_log_path         | /var/log/ntp                         | Location of the NTP log files.                                                        |
| **firewall_ntp_interfaces**   | empty                                | A list of interfaces to enable for the NTP server.                                    |
| firewall_ntp_pool             | North America NTP Pools              | List of pools to query upstream.                                                      |
| firewall_ntp_servers          | empty                                | A list of specific servers to query.                                                  |
| firewall_ntp_restrict_options | "kod limited nomodify nopeer notrap" | Restrict options for enabled interfaces.                                              |
| firewall_ntp_permit_networks  | empty                                | List of additional networks to permit for NTP.  Internal network are always included. |
| firewall_ntp_stats_enable     | false                                | Enable detailed logging for NTP peers.                                                |

Example Playbook
----------------

Main playbook:

```yaml
---
- name: Set up firewall
  hosts: firewall
  become: false
  vars_files:
    - vars/firewall.yml

  roles:
    - wesmarcum.firewall.base
    - wesmarcum.firewall.ntp
```

Define all variables in the `vars/firewall.yml` file:

```yaml
firewall_ntp_interfaces:
  - em0.2

firewall_ntp_permit_networks:
  - 172.20.1.0/24
  - fd00:0001:0002:0003::/64
```

License
-------

MIT

Author Information
------------------

https://github.com/wesmarcum/
