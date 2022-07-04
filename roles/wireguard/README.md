Firewall - Wireguard
====================

The `wireguard` role installs Wireguard as a kernel module or Go binary, then configures Wireguard tunnel interfaces defined in `firewall_wireguard_conf`.  No key management is provided, keys must be manually generated and passed in as configuration items.

An example remote access configuration is given below in the [Example Playbook](#example-playbook) section.  Site to site tunnels can also be easily configured by adjusting the IP address ranges on the sample configuration.

Installation
------------

See documentation for the collection `wesmarcum.firewall`.

Requirements
------------

No other roles are required.  Defining the variable `firewall_wireguard_conf` is required for configuration.

Role Variables
--------------

#### General:

| Variable Name               | Default Value              | Description                                                            |
|-----------------------------|----------------------------|------------------------------------------------------------------------|
| firewall_wireguard_enable   | true                       | Enable or disable Wireguard.                                           |
| firewall_wireguard_path     | "/usr/local/etc/wireguard" | Configuration directory.                                               |
| firewall_wireguard_use_kmod | false                      | Use kernel module, otherwise the Go implementation will be used.       |
| firewall_wireguard_env      | ""                         | Environment variables to pass.  Only applies to the Go implementation. |
| **firewall_wireguard_conf** | empty                      | List of Wireguard interfaces and parameters to configure.              |

#### Config:

The `firewall_wireguard_conf` variable is a list of configuration items for Wireguard tunnels.  Configuration parameters are listed in the table below.

| Parameter            | Example                                          | Description                                                                          |
|----------------------|--------------------------------------------------|--------------------------------------------------------------------------------------|
| **name**             | wg0                                              | Name of the tunnel interface to create.                                              |
| private_key          | "{{ vault_wireguard_wg0_private_key }}"          | Private key for the Wireguard interface.  Recommend redirecting to a vault variable. |
| listen_port          | 51820                                            | UDP port to listen on for incoming connections.                                      |
| address              | "172.20.0.1/24, fd00:0001:0002:0003::1/64"       | Addresses to assign to the WG tunnel interface.                                      |
| mtu                  | 1350                                             | MTU for the tunnel interface.                                                        |
| dns                  | "172.20.1.1, fd00:0001:0002:0004:1"              | DNS servers to use for the interface.                                                |
| table                | 1234                                             | Routing table where routes will be added.                                            |
| fwmark               | 0x12345678                                       | Optional 32-bit fwmark for outgoing packets.                                         |
| peers                |                                                  | A list of peers to configure.  Options are below.                                    |
| description          | "MacBook Pro"                                    | Helpful description for the peer.                                                    |
| **public_key**       | "{{ vault_wireguard_wg0_peer_public_key }}"      | Public key for the peer.  Can be redirected to a vault or listed in the config.      |
| preshared_key        | "{{ vault_wireguard_wg0_peer_preshared_key }}"   | Optional pre-shared key for authentication.  This should be stored in a vault.       |
| **allowed_ips**      | "172.20.0.10/32, fd00:0001:0002:0003::10/128"    | Allowed tunneled IPs for the peer.                                                   |
| endpoint             | "`vpn.example.com:51820`"                        | DNS name (or IP) and port to use to connect to the peer.                             |
| persistent_keepalive | 60                                               | Send keepalive packets every 60 seconds.  (0 = disabled)                             |

All keys for the configuration must be generated manually.  The Wireguard [quick start guide](https://www.wireguard.com/quickstart/) is a good reference for generating and managing keys.

Quick reference:
* Generate a public / private keypair: `wg genkey | tee privatekey | wg pubkey > publickey`
* Generate a pre-shared key: `wg genpsk`

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
    - wesmarcum.firewall.wireguard
```

Define all variables in the `vars/firewall.yml` file.  A remote access configuration can be configured as shown:

```yaml
firewall_wireguard_conf:
  - name: wg0
    private_key: "{{ vault_wireguard_wg0_private_key }}"
    listen_port: 51820
    address: "172.20.0.1/24, fd00:0001:0002:0003::1/64"
    mtu: 1350
    peers:
      - description: "Peer 1"
        public_key: "{{ vault_wireguard_wg0_peer1_public_key }}"
        preshared_key: "{{ vault_wireguard_wg0_peer1_preshared_key }}"
        allowed_ips: "172.20.0.10/32, fd00:0001:0002:0003::10/128"
        persistent_keepalive: 60
      - description: "Peer 2"
        public_key: "{{ vault_wireguard_wg0_peer2_public_key }}"
        preshared_key: "{{ vault_wireguard_wg0_peer2_preshared_key }}"
        allowed_ips: "172.20.0.11/32, fd00:0001:0002:0003::11/128"
        persistent_keepalive: 60
```

This will create the following configuration on the server (`/usr/local/etc/wireguard/<name>.conf`):

```
[Interface]
PrivateKey = <private key>
Address = 172.20.0.1/24, fd00:0001:0002:0003::1/64
ListenPort = 51820
MTU = 1350

# Peer 1
[Peer]
PublicKey = <public key>
PresharedKey = <pre-shared key>
AllowedIPs = 172.20.0.10/32, fd00:0001:0002:0003::10/128
PersistentKeepalive = 60

# Peer 2
[Peer]
PublicKey = <public key>
PresharedKey = <pre-shared key>
AllowedIPs = 172.20.0.11/32, fd00:0001:0002:0003::11/128
PersistentKeepalive = 60
```

A corresponding client configuration can be used:

```
[Interface]
PrivateKey = <private key>
Address = 172.20.0.10/32, fd00:0001:0002:0003::10/128
DNS = 172.20.0.1, fd00:0001:0002:0003::1
MTU = 1350

[Peer]
PublicKey = <public key>
PresharedKey = <preshared key>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = vpn.example.com:51820
PersistentKeepalive = 60
```

Once you have the client configuration ready, it can be directly copied or imported via QR code (`qrencode -t ansiutf8 < client_configuration`).

License
-------

MIT

Author Information
------------------

https://github.com/wesmarcum
