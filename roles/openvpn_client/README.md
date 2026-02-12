Firewall - OpenVPN Client
=========================

The `openvpn_client` role installs OpenVPN and configures one or more client configurations.  This will enable the firewall to connect as a client to an upstream VPN provider.  Routes are not installed to the routing table by default, but this behavior can be changed.  Traffic can be routed over VPN tunnels with the `route-to` directive in PF.

Installation
------------

See documentation for the collection `wesmarcum.firewall`.

Requirements
------------

No other roles are required.  The variable `firewall_openvpn_client_conf` is required to be defined.  This contains the main OpenVPN configuration parameters.

Role Variables
--------------

#### General:

| Variable Name                                  | Default Value            | Description                                                                           |
|------------------------------------------------|--------------------------|---------------------------------------------------------------------------------------|
| firewall_openvpn_client_enable                 | true                     | Enable or disable the OpenVPN client.                                                 |
| firewall_openvpn_client_path                   | "/usr/local/etc/openvpn" | Base directory for client configurations.                                             |
| firewall_openvpn_client_uid                    | 990                      | User ID for the OpenVPN user.                                                         |
| firewall_openvpn_client_gid                    | 990                      | Group ID for the OpenVPN user.                                                        |
| **firewall_openvpn_client_conf**               | empty                    | List of dictionary values for the client configurations.  Parameters are given below. |
| firewall_openvpn_client_verbosity              | 3                        | Logging verbosity.                                                                    |
| firewall_openvpn_client_drop_priv              | false                    | Drop privileges for the client process.  Can cause permissions issues.                |
| firewall_openvpn_client_chroot                 | false                    | Chroot OpenVPN process.  Can cause issues re-configuring the tunnel interface.        |
| firewall_openvpn_client_script_security        | 1                        | Script security level (0-3).                                                          |
| firewall_openvpn_client_keepalive_ping         | 10                       |                                                                                       |
| firewall_openvpn_client_keepalive_ping_restart | 60                       |                                                                                       |
| firewall_openvpn_client_retry                  | 5                        | Seconds to wait between connection attempts.                                          |
| firewall_openvpn_client_resolv_retry           | 30                       | If hostname resolution fails, wait this number of seconds before failing.             |
| firewall_openvpn_client_reneg                  | 3600                     | Renegotiate the data channel after this number of seconds.                            |

#### Config:

The `firewall_openvpn_client_conf` variable is a list of OpenVPN client configurations to deploy.  This can contain multiple configurations.  Each entry can utilize the parameters in the table below:

| Parameter             | Example                                            | Description                                                                                  |
|-----------------------|----------------------------------------------------|----------------------------------------------------------------------------------------------|
| **name**              | "my_provider"                                      | Name for the configuration/provider.  Lower case letters and underscores are recommended.    |
| interface             | `tun100`                                           | Interface name for the layer 3 tunnel.                                                       |
| **username**          | "{{ vault_openvpn_my_provider_username }}"         | Username for the VPN provider.                                                               |
| **password**          | "{{ vault_openvpn_my_provider_password }}"         | Password for the VPN provider.                                                               |
| ca_cert               | "files/usr/local/etc/openvpn/my_provider/ca.crt"   | Path to the CA certificate for the VPN provider.                                             |
| tls_key               | "files/usr/local/etc/openvpn/my_provider/tls.key" | Path to the TLS key for the VPN provider.                                                    |
| tls_crypt             | true                               | Enable tls-crypt |
| tls_auth              | true                               | Enable tls-auth  |
| **remotes**           | see examples below                                 | A list of remotes.  Keys should contain `host` (required), `port`, and `protocol`.           |
| remote_random         | false                                              | Randomize server on startup.  Otherwise the list will be processed top down.                 |
| protocol              | udp                                                | Default protocol for the client.                                                             |
| auth                  | SHA512                                             | Authentication for control and data channel packets.                                         |
| data_ciphers          | "CHACHA20-POLY1305:AES-256-GCM:AES-128-GCM"        | Allowed ciphers for negotiation.                                                             |
| data_ciphers_fallback | "AES-256-GCM"                                      | Cipher for fallback if negotiation fails.                                                    |
| route_nopull          | true                                               | Don't add routes to the routing table.                                                       |
| mtu                   | 1500                                               | Tunnel MTU                                                                                   |
| mtu_extra             | 32                                                 | Assume that the TUN/TAP device may return n number of bytes extra.                           |
| mssfix                | 1450                                               | Limit TCP send packet sizes such that a UDP encapsulated packet will not exceed this number. |

Note: Parameters in **bold** are required.

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
    - wesmarcum.firewall.openvpn_client
```

Define all variables in the `vars/firewall.yml` file.  Example with all options used:

```yaml
firewall_openvpn_client_conf:
  - name: my_provider
    tls_crypt: true
    tls_key: "files/usr/local/etc/openvpn/my_provider/tls-auth"
    ca_cert: "files/usr/local/etc/openvpn/my_provider/ca.crt"
    username: "{{ vault_openvpn_my_provider_username }}"
    password: "{{ vault_openvpn_my_provider_password }}"
    interface: tun100
    remotes:
      - host: host1.vpnprovider.net
        port: 1194
        protocol: udp
      - host: host2.vpnprovider.net
        port: 1194
        protocol: udp
    remote_random: false
    protocol: udp
    auth: SHA512
    data_ciphers: "CHACHA20-POLY1305:AES-256-GCM:AES-128-GCM"
    data_ciphers_fallback: "AES-256-GCM"
    route_nopull: true
    mtu: 1500
    mtu_extra: 32
    mssfix: 1450
```

Minimal configuration:

```yaml
firewall_openvpn_client_conf:
  - name: my_provider
    tls_crypt: true
    tls_key: "files/usr/local/etc/openvpn/my_provider/tls-auth"
    ca_cert: "files/usr/local/etc/openvpn/my_provider/ca.crt"
    username: "{{ vault_openvpn_my_provider_username }}"
    password: "{{ vault_openvpn_my_provider_password }}"
    interface: tun100
    remotes:
      - host: host1.vpnprovider.net
        port: 1194
        protocol: udp
```

License
-------

MIT

Author Information
------------------

https://github.com/wesmarcum/
