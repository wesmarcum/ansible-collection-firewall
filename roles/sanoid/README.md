Firewall - Sanoid
=================

The `sanoid` role installs and configures Sanoid for automatic ZFS snapshot management.  The role includes a default template (`templates/usr/local/etc/sanoid/sanoid_default.conf.j2`) for the Sanoid configuration file.  This configuration should be suitable for most situations.  You can provide your own configuration template by replacing `default` with the hostname of the target system.  For example, if the hostname is `myhost`, your configuration file should be named `templates/usr/local/etc/sanoid/sanoid_myhost.conf.j2`

Requirements
------------

No additional roles are required.

Role Variables
--------------

#### General:

| Variable Name                | Default Value         | Description                         |
|------------------------------|-----------------------|-------------------------------------|
| firewall_sanoid_enable       | true                  | Enable or disable Sanoid.           |
| firewall_sanoid_path         | /usr/local/etc/sanoid | Main path for Sanoid configuration. |
| firewall_sanoid_runtime_path | /var/run/sanoid       | Path for Sanoid runtime files.      |

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
    - wesmarcum.firewall.sanoid
```

Define all variables in the `vars/firewall.yml` file:

```yaml
firewall_sanoid_enable: true
```

License
-------

MIT

Author Information
------------------

https://github.com/wesmarcum/
