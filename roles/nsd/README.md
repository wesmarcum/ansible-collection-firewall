Firewall - NSD
==============

The `nsd` role installs and configures the [NSD](https://nlnetlabs.nl/projects/nsd) authoritative name server.  This role is designed to deploy a simple NSD instance that serves queries for local zones.  By default, NSD will be configured to listen on port `udp/53530` on localhost interfaces.  This configuration can function as an upstream authoritative name server for a recursive name server, such as [Unbound](https://nlnetlabs.nl/projects/unbound/).  This role works well with the `wesmarcum.firewall.unbound` role.

Installation
------------

See documentation for the collection `wesmarcum.firewall`.

Requirements
------------

Using the following roles are recommended:
- wesmarcum.firewall.base
- wesmarcum.firewall.unbound

Defining the variable `firewall_nsd_zones` is required for the role to define zone files and copy them to the target system.  Zone files must be defined in the `files/usr/local/etc/nsd/zones` directory before running this role.  The role will automatically copy the zone files to the target system.

Role Variables
--------------

#### General:

| Variable Name                  | Default Value                     | Description                                                                                   |
|--------------------------------|-----------------------------------|-----------------------------------------------------------------------------------------------|
| firewall_nsd_enable            | true                              | Enable or disable the NSD service.                                                            |
| firewall_nsd_username          | nsd                               | User the NSD service will run as.                                                             |
| firewall_nsd_path              | /usr/local/etc/nsd                | Base directory for NSD.                                                                       |
| firewall_nsd_server_count      | 2                                 | Number of NSD servers to fork.  Usually set to the number of CPUs.                            |
| firewall_nsd_zonefiles_check   | yes                               | Check mtime of zonefiles on start and SIGHUP.                                                 |
| **firewall_nsd_zones**         | empty                             | List of authoritative zones for NSD.                                                          |
| firewall_nsd_debug_mode        | no                                | Enable debug mode.  Does not fork daemon process into the background.                         |

#### Network:

| Variable Name                 | Default Value                              | Description                                                                             |
|-------------------------------|--------------------------------------------|-----------------------------------------------------------------------------------------|
| firewall_nsd_port             | 53530                                      | UDP port that the NSD service will listen on.                                           |
| firewall_nsd_interfaces       | ::1, 127.0.0.1                             | Interfaces to bind to.  Localhost only by default.  Use ::0 and 0.0.0.0 to specify all. |
| firewall_nsd_enable_ipv4      | yes                                        | Enable IPv4 for NSD.                                                                    |
| firewall_nsd_enable_ipv6      | yes                                        | Enable IPv6 for NSD.                                                                    |
| firewall_nsd_ip_transparent   | no                                         | Allow binding to non-local addresses.                                                   |
| firewall_nsd_ip_freebind      | no                                         | Allow binding to addresses that are down.                                               |
| firewall_nsd_tls_enable       | false                                      | Enable DoT for clients.  The path for a valid TLS key/cert must be specified.           |
| firewall_nsd_tls_port         | 853                                        | Port for serving DoT requests (if enabled).                                             |
| firewall_nsd_tls_interfaces   | empty                                      | List of interfaces to enable for DoT requests.                                          |
| firewall_nsd_tls_service_key  | "{{ firewall_nsd_path }}/cert/private.key" | Private key to use for DoT requests.                                                    |
| firewall_nsd_tls_service_pem  | "{{ firewall_nsd_path }}/cert/public.pem"  | Certificate to use for DoT requests.                                                    |
| firewall_nsd_tls_service_ocsp | empty                                      | OCSP pem file for the DoT service.  For OCSP stapling.                                  |

#### Logging:

| Variable Name                  | Default Value    | Description                                  |
|--------------------------------|------------------|----------------------------------------------|
| firewall_nsd_logging_verbosity | 2                | Verbosity level for logging.                 |
| firewall_nsd_logfile           | /var/log/nsd.log | Logfile for NSD.                             |
| firewall_nsd_log_only_syslog   | no               | Log only to Syslog.                          |
| firewall_nsd_log_time_ascii    | yes              | Log timestamp in ascii (`y-m-d h:m:s.msec`). |

#### Security:

| Variable Name                  | Default Value                              | Description                                                                                   |
|--------------------------------|--------------------------------------------|-----------------------------------------------------------------------------------------------|
| firewall_nsd_refuse_any        | yes                                        | Refuse queries of type ANY.                                                                   |
| firewall_nsd_hide_identity     | yes                                        | Refuse id.server and hostname.bind queries.                                                   |
| firewall_nsd_hide_version      | yes                                        | Refuse version.server and version.bind queries.                                               |
| firewall_nsd_secret            | "{{ vault_firewall_nsd_secret }}"          | Secret for zone transfers.  Randomly generated unless `vault_firewall_nsd_secret` is defined. |
| firewall_nsd_secret_hash       | "hmac-sha256"                              | Hash type for the secret.                                                                     |
| firewall_nsd_drop_updates      | no                                         | Drop UPDATE queries.                                                                          |
| firewall_nsd_version           | "DNS"                                      | Version string the server responds to for chaos queries.                                      |
| firewall_nsd_identity          | "DNS"                                      | Identity for CH TXT ID.SERVER.                                                                |
| firewall_nsd_nsid              | ""                                         | NSID identity.  Can be a hex or ascii string.                                                 |
| firewall_nsd_minimal_responses | no                                         | If enabled, only emit extra data for referrals.                                               |
| firewall_nsd_confine_to_zone   | no                                         | Do not return additional info if the apex zone does not match the apex zone of the query.     |

#### Performance:

| Variable Name                        | Default Value | Description                                                                  |
|--------------------------------------|---------------|------------------------------------------------------------------------------|
| firewall_nsd_reuseport               | no            | Use SO_REUSEPORT socket option for performance.                              |
| firewall_nsd_send_buffer             | 1048576       | Max socket send buffer size in bytes.                                        |
| firewall_nsd_receive_buffer          | 1048576       | Max socket receive buffer size in bytes.                                     |
| firewall_nsd_tcp_count               | 100           | Max concurrent TCP connections per server.                                   |
| firewall_nsd_tcp_reject_overflow     | no            | Immediately close TCP connections after the max connection limit is reached. |
| firewall_nsd_tcp_query_count         | 0             | Max queries served on a single TCP connection. 0 = unlimited.                |
| firewall_nsd_tcp_timeout             | 120           | TCP timeout in seconds.                                                      |
| firewall_nsd_tcp_mss                 | 0             | TCP MSS.  0 = use system default.                                            |
| firewall_nsd_tcp_outgoing_tcp_mss    | 0             | TCP MSS for outgoing AXFR request.  0 = use system default.                  |
| firewall_nsd_ipv4_edns_size          | 1232          | EDNS buffer size for IPv4.                                                   |
| firewall_nsd_ipv6_edns_size          | 1232          | EDNS buffer size for IPv6.                                                   |
| firewall_nsd_xfrd_reload_timeout     | 1             | Seconds between reloads triggered by xfrd.                                   |
| firewall_nsd_rrl_size                | 1000000       | Size of the hashtable for response rate limiting.                            |
| firewall_nsd_rrl_ratelimit           | 200           | Max queries per second per source. 0 = disabled.                             |
| firewall_nsd_rrl_slip                | 2             | Number of packets to discard before sending a SLIP response.                 |
| firewall_nsd_rrl_ipv4_prefix_length  | 24            | IPv4 prefix length.  Addresses are grouped by netblock.                      |
| firewall_nsd_rrl_ipv6_prefix_length  | 64            | IPv6 prefix length.  Addresses are grouped by netblock.                      |
| firewall_nsd_rrl_whitelist_ratelimit | 2000          | Max queries per second for whitelisted types.                                |
| firewall_nsd_round_robin             | no            | Use round robin rotation of answers.                                         |
| firewall_nsd_zonefiles_write         | 3600          | Write changed zonefiles to disk, every N seconds.                            |

#### Statistics:

| Variable Name                    | Default Value | Description                                                       |
|----------------------------------|---------------|-------------------------------------------------------------------|
| firewall_nsd_statistics_interval | 3600          | Number of seconds between printing stats to the log. 0 = disabled |

#### Zones:

The `firewall_nsd_zones` variable should be a list of internal zones.  Each zone in the list can have the following parameters:

| Parameter | Example                                                | Description                                                                         |
|-----------|--------------------------------------------------------|-------------------------------------------------------------------------------------|
| name      | "`lan.mydomain.com`" or "1.0.10.in-addr.arpa"            | DNS zone name, forward or reverse.                                                  |
| zonefile  | "lan.mydomain.com.zone" or "lan.mydomain.com.ipv4.rev" | Name of the zone file to copy.  Zones should be in `files/usr/local/etc/nsd/zones`. |

#### DNS over TLS:

DoT can be enabled by setting the `firewall_nsd_tls_enable` variable to `true`.  You **must** have a valid certificate and private key defined by the `firewall_nsd_tls_service_pem` and `firewall_nsd_tls_service_key` path variables.  This role does not automatically generate certificates.  The [community.crypto](https://docs.ansible.com/ansible/latest/collections/community/crypto/index.html#plugins-in-community-crypto) Ansible modules can do this for you in separate tasks.  You must also specify a list of interfaces to enable for DoT:

```yaml
firewall_nsd_tls_enable: true
firewall_nsd_tls_interfaces:
  - em0
```  

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
    - wesmarcum.firewall.nsd
```

Define all variables in the `vars/firewall.yml` file:

```yaml
firewall_nsd_enable: true

firewall_nsd_zones:
  - name: "lan.mydomain.com"
    zonefile: "lan.mydomain.com.zone"
  - name: "1.0.10.in-addr.arpa"
    zonefile: "lan.mydomain.ipv4.rev"
  - name: "0.0.0.0.2.0.0.0.1.0.0.0.0.0.d.f.ip6.arpa"
    zonefile: "lan.mydomain.com.ipv6.rev"
```

License
-------

MIT

Author Information
------------------

https://github.com/wesmarcum/
