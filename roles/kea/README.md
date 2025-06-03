Firewall - Kea
==============

The `kea` role installs and configures the [kea](https://kea.isc.org/) package for DHCPv4 and DHCPv6.

Installation
------------

See documentation for the collection `wesmarcum.firewall`.

Requirements
------------

Using the role `wesmarcum.firewall.base` is recommended.  Defining the variables `firewall_kea_dhcp4_config` and `firewall_kea_dhcp6_config` are required if you are using both DHCPv4 and DHCPv6.

Role Variables
--------------

#### General

| Variable Name                 | Default Value            | Description                               |
|-------------------------------|--------------------------|-------------------------------------------|
| firewall_kea_enable           | true                     | Enable or disable the main kea service.   |
| firewall_kea_dhcp4_enable     | true                     | Enable the DHCPv4 server.                 |
| firewall_kea_dhcp6_enable     | true                     | Enable the DHCPv6 server.                 |
| firewall_kea_path             | "/usr/local/etc/kea"     | Location for the kea configuration files. |
| firewall_kea_lease_path       | "/var/db/kea" | Location for the DHCP lease files.        |
| firewall_kea_log_path       | "/var/log/kea" | Location for kea log files.        |
| **firewall_kea_dhcp4_config** | empty                    | DHCPv4 server configuration.              |
| **firewall_kea_dhcp6_config** | empty                    | DHCPv6 server configuration.              |

#### Config

The `firewall_kea_dhcp4_config` and `firewall_kea_dhcp6_config` variables define the configuration for each server.  These variables are YAML dictionaries that are directly converted to JSON for the Kea configuration files.  An example is included below.  Please see the Kea [documentation](https://kea.readthedocs.io/en/latest/) for all options.

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
    - wesmarcum.firewall.kea
```

Define all variables in the `vars/firewall.yml` file:

```yaml
# DHCPv4 configuration for Kea.  The sample configuration below is formatted to be converted directly to JSON.  Please see the documentation
# at kea.isc.org for all options.  For a quick start, uncomment the appropriate lines and change values to match your network.
firewall_kea_dhcp4_config:
  Dhcp4:
    # interfaces to activate for Kea
    interfaces-config:
      interfaces:
        - "*"
        # - em1
        # - em2
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
      name: /var/db/kea/dhcp4.leases
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
    # Change these subnets to match your configuration.
    subnet4:
      - subnet: 192.168.0.0/24
        id: 1000
        interface: em1
        pools:
          - pool: 192.168.0.100 - 192.168.0.200
        option-data:
          - name: routers
            data: 192.168.0.1
          - name: domain-name
            data: example.local
          - name: domain-name-servers
            data: 192.168.0.1
        reservations:
          - hw-address: '00:11:22:33:aa:bb'
            ip-address: 192.168.0.10
            hostname: myhost
      - subnet: 10.0.0.0/24
        id: 1001
        interface: em2
        pools:
          - pool: 10.0.0.100 - 10.0.0.200
        option-data:
          - name: routers
            data: 10.0.0.1
          - name: domain-name
            data: example2.local
          - name: domain-name-servers
            data: 10.0.0.1
    loggers:
      - name: kea-dhcp4
        output_options:
          - output: /var/log/kea/kea-dhcp4.log
            pattern: |
              {% raw %}%d{%j %H:%M:%S.%q} %c %m{% endraw %}
            flush: true
            maxsize: 10240000
            maxver: 8
        severity: INFO
        debuglevel: 0

# DHCPv6 configuration for Kea.  The sample configuration below is formatted to be converted directly to JSON.  Please see the documentation
# at kea.isc.org for all options.  For a quick start, uncomment the appropriate lines and change values to match your network.
firewall_kea_dhcp6_config:
  Dhcp6:
    interfaces-config:
      interfaces:
        - "*"
        # - em1
        # - em2
  # control socket - uncomment if api is enabled
  # control-socket:
    # socket-type: unix
    # socket-name: "/tmp/kea4-ctrl-socket"
    lease-database:
      type: memfile
      persist: true
      lfc-interval: 3600
      name: /var/db/kea/dhcp6.leases
    expired-leases-processing:
      reclaim-timer-wait-time: 10
      flush-reclaimed-timer-wait-time: 25
      hold-reclaimed-time: 3600
      max-reclaim-leases: 100
      max-reclaim-time: 250
      unwarned-reclaim-cycles: 5
    renew-timer: 10800
    rebind-timer: 21600
    valid-lifetime: 43200
    max-valid-lifetime: 86400
    min-valid-lifetime: 3600
    # Change these subnets to match your configuration.
    subnet6:
      - subnet: 'fd00:2001:0001::/64'
        id: 2000
        interface: em1
        pools:
          - pool: 'fd00:2001:0001::1000 - fd00:2001:0001::2000'
        option-data:
          - name: domain-search
            data: example.local
          - name: dns-servers
            data: 'fd00:2001:0001::1'
        reservations:
          - duid: "00:01:00:01:00:00:00:00:00:00:00:00:00:11"
            ip-addresses:
              - fd00:2001:0001::10
            hostname: "myhost.example.local"
    loggers:
      - name: kea-dhcp6
        output_options:
          - output: /var/log/kea/kea-dhcp6.log
            pattern: |
              {% raw %}%d{%j %H:%M:%S.%q} %c %m{% endraw %}
            flush: true
            maxsize: 10240000
            maxver: 8
        severity: INFO
        debuglevel: 0
```

License
-------

MIT

Author Information
------------------

<https://github.com/wesmarcum/>
