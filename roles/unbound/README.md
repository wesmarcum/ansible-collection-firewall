Firewall - Unbound
==================

The `unbound` role installs and configures the [Unbound](https://nlnetlabs.nl/projects/unbound/) recursive, caching DNS resolver.  By default, Unbound will be configured to forward DNS requests to upstream servers defined with `firewall_unbound_forward_zones` (via DoT).  The upstream servers can be changed, or the root servers can be queried directly.  Local zones can be defined to provide local DNS resolution with local data or an upstream authoritative server (such as [NSD](https://nlnetlabs.nl/projects/nsd/)).  This role works well with the `wesmarcum.firewall.nsd` role to provide full DNS services.

Unbound can also be configured to function as a DNS filter with dynamic or static blocking.

Installation
------------

See documentation for the collection `wesmarcum.firewall`.

Requirements
------------

The role `wesmarcum.firewall.base` is required.  The Unbound role uses the `firewall_interfaces` variable to automatically generate configuration items in `unbound.conf`.

Unbound has **many** configuration options.  Most of the options listed below are not mandatory - they are designed to be tunable depending on your environment.  For a quick start, see the configuration examples.

Role Variables
--------------

#### General:

| Variable Name                        | Default Value          | Description                                                                                    |
|--------------------------------------|------------------------|------------------------------------------------------------------------------------------------|
| firewall_unbound_enable              | true                   | Enable or disable unbound.                                                                     |
| firewall_unbound_username            | unbound                | User the Unbound service will run as.                                                          |
| firewall_unbound_path                | /usr/local/etc/unbound | Base directory for Unbound.                                                                    |
| firewall_unbound_module_config       | "validator iterator"   | Unbound modules to activate.  DNSSEC and recursive resolver are used by default.               |
| firewall_unbound_max_ttl             | 86400                  | Max TTL for messages in the cache.                                                             |
| firewall_unbound_min_ttl             | 300                    | Min TTL for messages in the cache.                                                             |
| firewall_unbound_edns_buffer_size    | 1232                   | Number of bytes to advertise as the EDNS buffer size.                                          |
| firewall_unbound_rrset_roundrobin    | yes                    | Rotates RRSet order in response.                                                               |
| firewall_unbound_private_domains     | empty                  | List of private domains.  Allows responses with private IP addresses.                          |
| firewall_unbound_acl_networks        | empty                  | List of additional networks to allow for DNS queries.  Internal networks are already included. |
| firewall_unbound_local_zones         | empty                  | List of local zones.  DNSSEC will be disabled for these zones.  See examples below.            |
| firewall_unbound_local_zones_rev     | empty                  | List of local zones for reverse lookups.  Defined the same as `firewall_unbound_local_zones`   |
| firewall_unbound_local_zone_template | true                   | If `true`, generate local/zone data with a template.  Otherwise copy from local machine.       |
| firewall_unbound_stub_zones          | empty                  | List of stub zones to forward to another resolver, can be used for a local NSD instance.       |
| firewall_unbound_stub_default_port   | 53530                  | Default port to forward for upstream resolver.                                                 |
| firewall_unbound_forward_enable      | true                   | Enable forwarding to upstream resolvers.  If `false`, use root servers.                        |
| firewall_unbound_forward_zones       | quad9                  | List of forward zones and resolvers.  See examples below.                                      |
| firewall_unbound_dynamic_block       | false                  | Enable or disable dynamic blocking for ads/malware.                                            |
| firewall_unbound_dynamic_block_url   | StevenBlack Hosts      | URL to pull host block list.  Updated nightly by default.                                      |

#### Network:

| Variable Name                       | Default Value                                  | Description                                 |
|-------------------------------------|------------------------------------------------|---------------------------------------------|
| firewall_unbound_port               | 53                                             | UDP port to listen on.                      |
| firewall_unbound_enable_ipv4        | true                                           | Enable IPv4 for Unbound.                    |
| firewall_unbound_enable_ipv6        | true                                           | Enable IPv6 for Unbound.                    |
| firewall_unbound_enable_udp         | true                                           | Enable listening on UDP.                    |
| firewall_unbound_enable_tcp         | true                                           | Enable listening on TCP.                    |
| firewall_unbound_prefer_ipv6        | yes                                            | Prefer IPv6 instead of IPv4.                |
| firewall_unbound_tls_enable         | false                                          | Enable TLS for DoT / DoH.                   |
| firewall_unbound_tls_service_key    | "{{ firewall_unbound_path }}/cert/private.key" | Private key for DoT / DoH requests.         |
| firewall_unbound_tls_service_pem    | "{{ firewall_unbound_path }}/cert/public.pem"  | Certificate for DoT / DoH requests.         |
| firewall_unbound_tls_doh_interfaces | empty                                          | List of interfaces to enable for DoH.       |
| firewall_unbound_tls_doh_port       | 443                                            | Default port to listen on for DoH requests. |
| firewall_unbound_tls_dot_interfaces | empty                                          | List of interfaces to enable for DoT.       |
| firewall_unbound_tls_dot_port       | 853                                            | Default port to listen on for DoT requests. |

#### Logging:

| Variable Name                           | Default Value | Description                                                     |
|-----------------------------------------|---------------|-----------------------------------------------------------------|
| firewall_unbound_logging_verbosity      | 2             | Logging verbosity (0-5).  2 = detailed operational information. |
| firewall_unbound_logfile                | ""            | Logfile to use (ignored when syslog is enabled).                |
| firewall_unbound_use_syslog             | yes           | Send logging to syslog instead of the logfile.                  |
| firewall_unbound_log_tag_queryreply     | yes           | Print "query" and "reply" with log messages.                    |
| firewall_unbound_log_local_zone_actions | no            | Do not print log lines to inform about local zone actions.      |
| firewall_unbound_log_queries            | no            | Do not print one line per query to the log.                     |
| firewall_unbound_log_replies            | no            | Do not print one line per reply to the log.                     |
| firewall_unbound_log_servfail           | no            | Do not log lines that say why queries return SERVFAIL.          |

#### Privacy:

| Variable Name                    | Default Value | Description                                                                                                         |
|----------------------------------|---------------|---------------------------------------------------------------------------------------------------------------------|
| firewall_unbound_aggressive_nsec | yes           | RFC 8198.  Use cached NSEC records to generate negative answers within a range and positive answers from wildcards. |
| firewall_unbound_delay_close     | 10000         | Extra delay for timeouted UDP ports before they are closed.                                                         |
| firewall_unbound_dnq_localhost   | no            | Add localhost to the do not query list.                                                                             |
| firewall_unbound_neg_cache_size  | 4M            | Bytes of the size for the negative cache.                                                                           |
| firewall_unbound_qname_min       | yes           | Send minimum amount of information to upstream servers.                                                             |

#### Security:

| Variable Name                             | Default Value                                  | Description                                                                                     |
|-------------------------------------------|------------------------------------------------|-------------------------------------------------------------------------------------------------|
| firewall_unbound_deny_any                 | yes                                            | Deny queries of type ANY with empty response.                                                   |
| firewall_unbound_harden_algo_downgrade    | yes                                            | Harden against algorithm downgrade when multiple are listed in the DS record.                   |
| firewall_unbound_harden_below_nxdomain    | yes                                            | Return nxdomain to queries for a name below another name known to be nxdomain.                  |
| firewall_unbound_harden_dnssec_stripped   | yes                                            | Require DNSSEC data for trust-anchored zones.                                                   |
| firewall_unbound_harden_glue              | yes                                            | Only trust glue if it is within the servers authority.                                          |
| firewall_unbound_harden_large_queries     | yes                                            | Ignore very large queries.                                                                      |
| firewall_unbound_harden_referral_path     | no                                             | Perform additional queries for infrastructure data to harden the referral path.  Experimental.  |
| firewall_unbound_harden_short_bufsize     | yes                                            | Ignore very small EDNS buffer sizes from queries.                                               |
| firewall_unbound_hide_identity            | yes                                            | Refuse id.server and hostname.bind queries.                                                     |
| firewall_unbound_hide_version             | yes                                            | Refuse version.server and version.bind queries.                                                 |
| firewall_unbound_identity                 | "DNS"                                          | Report this identity rather than the hostname of the server.                                    |
| firewall_unbound_ratelimit                | 1000                                           | Number of queries per second that can be sent for recursion.  Does not affect cached responses. |
| firewall_unbound_unwanted_reply_threshold | 10000                                          | Total number of replies to keep track of in every thread.                                       |
| firewall_unbound_use_caps_for_id          | yes                                            | Use random bits in the query to foil spoof attempts.  Experimental.                             |
| firewall_unbound_val_clean_additional     | yes                                            | Remove data from the additional section that are not signed properly.                           |

#### Performance:

| Variable Name                           | Default Value | Description                                                                     |
|-----------------------------------------|---------------|---------------------------------------------------------------------------------|
| firewall_unbound_infra_cache_slabs      | 4             | Number of slabs in the infra cache.  Reduces lock contention.                   |
| firewall_unbound_incoming_num_tcp       | 10            | Number of TCP buffers to allocate per thread.                                   |
| firewall_unbound_key_cache_slabs        | 4             | Number of slabs in the key cache.  Reduces lock contention.                     |
| firewall_unbound_msg_cache_size         | 50m           | Bytes size of the message cache.                                                |
| firewall_unbound_msg_cache_slabs        | 4             | Number of slabs in the message cache.  Reduces lock contention between threads. |
| firewall_unbound_num_queries_per_thread | 4096          | Number of queries that each thread will service simultaneously.                 |
| firewall_unbound_num_threads            | 4             | Number of threads to create to serve clients.                                   |
| firewall_unbound_outgoing_range         | 8196          | File descriptors per thread.                                                    |
| firewall_unbound_rrset_cache_size       | 100m          | Bytes of RRset cache.  Should be about 2x cache size.                           |
| firewall_unbound_rrset_cache_slabs      | 4             | Number of slabs in the rrset cache.                                             |
| firewall_unbound_minimal_responses      | yes           | Do not insert additional/authority sections in response when not needed.        |
| firewall_unbound_prefetch               | yes           | Message cache is prefetched before expiring to keep the cache up to date.       |
| firewall_unbound_prefetch_key           | yes           | Fetch DNSKEYs earlier in the validation process.                                |
| firewall_unbound_serve_expired          | no            | Serve old responses from cache while it is refreshed in the background.         |
| firewall_unbound_so_reuseport           | yes           | Open dedicated listening sockets for incoming queries.                          |
| firewall_unbound_jostle_timeout         | 200           | Timeout used when the server is very busy.                                      |
| firewall_unbound_infra_host_ttl         | 900           | TTL for entries in the host cache.                                              |
| firewall_unbound_infra_cache_numhosts   | 10000         | Number of hosts for which information is cached.                                |
| firewall_unbound_outgoing_num_tcp       | 10            | Number of outgoing TCP buffers to allocate per thread.                          |

#### Statistics:

| Variable Name                          | Default Value | Description                                                        |
|----------------------------------------|---------------|--------------------------------------------------------------------|
| firewall_unbound_statistics_interval   | 0             | Number of seconds between printing stats to the log.  0 = disabled |
| firewall_unbound_extended_statistics   | yes           | Print extended stats from `unbound-control`.                       |
| firewall_unbound_statistics_cumulative | yes           | If enable, stats are cumulative since starting unbound.            |

#### Local zones:

Local zones can be defined using the `firewall_unbound_local_zones` and `firewall_unbound_local_zones_rev` variables.  Local zones set the Unbound zone type - see [unbound.conf](https://unbound.docs.nlnetlabs.nl/en/latest/manpages/unbound.conf.html) for details on zone types.  DNSSEC is also disabled for local zones.  Parameters for these variables can be found in the table below.

| Parameter | Example                                       | Description                                                  |
|-----------|-----------------------------------------------|--------------------------------------------------------------|
| **zone**  | lan.local or "10.in-addr.arpa." (for reverse) | Name of the local zone.                                      |
| type      | nodefault                                     | Set the 'type' of zone for Unbound.  Default is 'nodefault'. |

Local zones will be generated automatically by default using the template `templates/usr/local/etc/unbound/local_zone.conf.j2` and the two local zone variables listed above.  If you wish to change this behavior, you can set `firewall_unbound_local_zone_template` to false.  In this scenario, you must provide your own custom local zone file at the location `files/usr/local/etc/unbound/local_zone.conf`.

#### Stub zones:

Zones that should be forwarded to a specific resolver, such as NSD, can be defined with `firewall_unbound_stub_zones`:

| Parameter   | Example    | Description                                                                                                                              |
|-------------|------------|------------------------------------------------------------------------------------------------------------------------------------------|
| **zone**    | lan.local. | Name of the zone.                                                                                                                        |
| nameservers |            | List of nameservers to use for the zone.  If not defined, localhost will be used with `firewall_unbound_stub_default_port` for the port. |
| ip          | 10.1.1.1   | IP of the DNS server.                                                                                                                    |
| port        | 53         | Port to send requests for the DNS server.                                                                                                |

#### Forward zones:

If `firewall_unbound_forward_enable` is `true`, you can define `firewall_unbound_forward_zones` to specify upstream zones and forwarder configuration:

| Parameter     | Example         | Description                                                                                            |
|---------------|-----------------|--------------------------------------------------------------------------------------------------------|
| **zone**      | "."             | Zone name.  This is usually set to root ("."), but this can be any zone name.                          |
| tls           | true            | Enable DoT for the resolvers.                                                                          |
| forward_first | false           | If this is enabled, Unbound will fall back to normal name resolution if the resolver returns SERVFAIL. |
| resolvers     |                 | List of resolvers to use for the zone.                                                                 |
| ip_address    | 9.9.9.9         | IP address for the resolver.  Can be IPv4 or IPv6.                                                     |
| port          | 853             | Upstream port to use for the resolver.                                                                 |
| tls_name      | `dns.quad9.net` | If DoT is enabled, this name will be verified in the certificate.                                      |

Note that if `firewall_unbound_forward_enable` is `false`, Unbound will query the root servers.

#### Dynamic blocking:

The `unbound` role can automatically set up dynamic blocking if `firewall_unbound_dynamic_block` is set to `true`.  By default, Steven Black's [hosts list](https://github.com/StevenBlack/hosts) is used.  If you'd like to change the list used, set `firewall_unbound_dynamic_block_url` to a URL containing a raw list of hosts - e.g. "https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts".  You can use any URL that provides host format.  Lists will be automatically updated each night via Cron.  A provided shell script translates the list from "hosts" format to Unbound's format for local data.

#### Static blocking:

If you wish to always block certain domains, these can be entered into a static block list.  The Unbound role looks for the file `files/usr/local/etc/unbound/static_block.conf` to copy to the target system.  The static block file will automatically be copied if it exists.  This file should be in Unbound's local-zone/local-data format:
```
server:
	local-zone: "example.com" redirect
	local-data: "example.com A 0.0.0.0"
        local-data: "example.com AAAA ::"

        local-zone: "anotherdomain.com" redirect
	local-data: "anotherdomain.com A 0.0.0.0"
        local-data: "anotherdomain.com AAAA ::"
```

#### DNS over HTTPS / DNS over TLS:

DoH / DoT can be enabled with `firewall_unbound_tls_enable`.  You **must** have a valid certificate and private key defined by the `firewall_unbound_tls_service_pem` and `firewall_unbound_tls_service_key` path variables.  This role does not automatically generate certificates.  The [community.crypto](https://docs.ansible.com/ansible/latest/collections/community/crypto/index.html#plugins-in-community-crypto) Ansible modules can do this for you in separate tasks.  You must also specify a list of interfaces to enable for DoT / DoH:

```yaml
firewall_unbound_tls_enable: true
firewall_unbound_tls_doh_interfaces:
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
    - wesmarcum.firewall.unbound
```

Define all variables in the `vars/firewall.yml` file:

```yaml
# Define private domains.
# This allows DNS responses with private addresses.
firewall_unbound_private_domains:
  - plex.tv
  - plex.direct

# Permit additional networks to query Unbound.
# Note that internal networks are automatically permitted.
firewall_unbound_acl_networks:
  - 172.16.1.0/24
  - fd0b:0001:0002:0003::/64

# Define a list of local zones and zone types.
# DNSSEC will be disabled for these zones.
firewall_unbound_local_zones:
  - zone: lan.local.
    type: nodefault
  - zone: guest.local.

# Define a list of local zones for reverse lookups.
firewall_unbound_local_zones_rev:
  - zone: "10.in-addr.arpa."
  - zone: "d.f.ip6.arpa."

# Stub zones for forwarding to a specific nameserver.
# This can be used for forwarding queries for local zones
# to a local NSD resolver.
firewall_unbound_stub_zones:
  - zone: lan.local.
    nameservers:
      - ip: ::1
        port: 53530
      - ip: 127.0.0.1
        port: 53530
  - zone: guest.local.
  - zone: "10.in-addr.arpa."
  - zone: "d.f.ip6.arpa."

# Forward zones and resolvers.  This is usually used to
# specify upstream DNS servers for the root zone.
firewall_unbound_forward_zones:
  - zone: "."
    tls: true
    forward_first: false
    resolvers:
      - ip_address: 9.9.9.9
        port: 853
        tls_name: dns.quad9.net
      - ip_address: 149.112.112.112
        port: 853
        tls_name: dns.quad9.net
      - ip_address: 2620:fe::fe
        port: 853
        tls_name: dns.quad9.net
      - ip_address: 2620:fe::9
        port: 853
        tls_name: dns.quad9.net

# Enable dynamic blocking for malware/ads.
firewall_unbound_dynamic_block: true
```

License
-------

MIT

Author Information
------------------

https://github.com/wesmarcum
