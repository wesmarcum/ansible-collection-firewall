Firewall - inadyn
=================

The `inadyn` role installs and configures the [inadyn](https://github.com/troglobit/inadyn) package for dynamic dns updates.  inadyn supports many providers out of the box, including custom configurations.

Installation
------------

See documentation for the collection `wesmarcum.firewall`.

Requirements
------------

Using the role `wesmarcum.firewall.base` is recommended.  Defining the variable `firewall_inadyn_conf` is required for this role to function.

Role Variables
--------------

#### General:

| Variable Name               | Default Value                | Description                                                                                      |
|-----------------------------|------------------------------|--------------------------------------------------------------------------------------------------|
| firewall_inadyn_enable      | true                         | Enable or disable inadyn.                                                                        |
| firewall_inadyn_conf_file   | "/usr/local/etc/inadyn.conf" | Location of the configuration file.                                                              |
| firewall_inadyn_period      | 300                          | Interval in seconds to check the external IP.                                                    |
| firewall_inadyn_user_agent  | empty                        | Set a custom user agent.  If empty, inadyn uses "inadyn/VERSION".                                |
| firewall_inadyn_interface   | empty                        | Set the external interface to track.  If not set, a check IP service is used.                    |
| firewall_inadyn_allow_ipv6  | true                         | Enable AAAA record updates.                                                                      |
| **firewall_inadyn_conf**    | empty                        | List of dictionary values with inadyn configuration.  This should be used for builtin providers. |

#### inadyn conf:

The `firewall_inadyn_conf` variable is a list of inadyn providers.  You must provide configuration for at least one provider.  Custom configurations are also supported by using the "type" parameter.  List of parameters:

| inadyn Parameter | Examples                                     | Description                                                                                              |
|------------------|----------------------------------------------|----------------------------------------------------------------------------------------------------------|
| **name**         | `cloudflare.com:1`                           | Friendly name for the provider.  If you have multiple entries at the same provider, index with :#.       |
| type             | provider or custom                           | Specify a built-in provider or custom configuration.   Defaults to "provider".                           |
| ssl              | true                                         | Use SSL/TLS.  Set to 'true' by default.                                                                  |
| checkip_server   | 1.1.1.1                                      | Override the checkip server.  Default is `api.ipify.org`.                                                |
| checkip_path     | /cdn-cgi/trace                               | Set the path for the checkip server.                                                                     |
| checkip_ssl      | true                                         | Use SSL/TLS for the checkip service.  Default is 'true'.                                                 |
| checkip_command  | "/sbin/ifconfig eth0 ..."                    | Custom command to get the external IP from the system.                                                   |
| **username**     | myuser                                       | Provider username.  For Cloudflare, use the DNS zone name.                                               |
| **password**     | mypass                                       | Provider password.  For Cloudflare, use your API key.                                                    |
| **hostname**     | `myhost.example.com`                         | Hostname to update for external IP address.                                                              |
| ttl              | 1                                            | Set optional ttl.                                                                                        |
| proxied          | false                                        | Enable 'proxied' traffic for Cloudflare.                                                                 |
| user_agent       | Mozilla/5.0                                  | Set a custom user agent.  Required for some providers.                                                   |
| ddns_server      | `members.dyndns.org`                         | Dynamic DNS server for custom provider.                                                                  |
| ddns_path        | `/nic/update?hostname=%h.dyndns.org&myip=%i` | Dynamic DNS server path for custom provider.                                                             |
| ddns_response    | "OK" or { Arrr, kilroy }                     | A single string or list of strings to look for in a successful response. Default is "good", "OK" ...etc. |

Note: Parameters in **bold** are required.

To configure a built-in provider for Cloudflare:

```yaml
firewall_inadyn_conf:
  - name: "cloudflare.com:1"
    ssl: true
    checkip_server: 1.1.1.1
    checkip_path: /cdn-cgi/trace
    checkip_ssl: true
    username: "zone.name"
    password: "api_key"
    hostname: "myhost.example.com"
    ttl: 1
    proxied: false
```

You can also configure a custom configuration by using `type: custom`:

```yaml
firewall_inadyn_conf:
  - name: "namecheap"
    type: custom
    username: "myuser"
    password: "mypass"
    ddns_server: "dynamicdns.park-your-domain.com"
    ddns_path: "/update?domain=%u&password=%p&host=%h"
    hostname: "{ "@", "www", "test" }"
```

See the [inadyn github](https://github.com/troglobit/inadyn) page for a full list of examples.

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
    - wesmarcum.firewall.inadyn
```

Define all variables in the `vars/firewall.yml` file:

```yaml
firewall_inadyn_conf:
  - name: "cloudflare.com:1"
    checkip_server: 1.1.1.1
    checkip_path: /cdn-cgi/trace
    username: "zone.name"
    password: "api_key"
    hostname: "myhost.example.com"
    ttl: 1
    proxied: false
  - name: "cloudflare.com:2"
    checkip_server: dns64.cloudflare-dns.com
    checkip_path: /cdn-cgi/trace
    username: "zone.name"
    password: "api_key"
    hostname: "myhost.example.com"
    ttl: 1
    proxied: false
```

License
-------

MIT

Author Information
------------------

https://github.com/wesmarcum/
