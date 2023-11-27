Firewall - Monitoring
=====================

The `monitoring` role installs common monitoring packages and optionally configures the following:
  - Email reporting.
  - Interface bandwidth statistics with [vnstat](https://humdi.net/vnstat/).

Monitoring packages installed by default:
  - bmon
  - bottom
  - btop
  - gping
  - iftop
  - vnstat

Installation
------------

See documentation for the collection `wesmarcum.firewall`.

Requirements
------------

Using the role `wesmarcum.firewall.base` is recommended.  If you want to install basic monitoring packages, simply include the role `wesmarcum.firewall.monitoring` in your playbook.  Periodic email summaries can be configured with the options below.

Role Variables
--------------

#### General:

| Variable Name                               | Default Value                | Description                                                          |
|---------------------------------------------|------------------------------|----------------------------------------------------------------------|
| firewall_monitoring_packages                | see "defaults/main.yml"      | Default packages to install.                                         |
| firewall_monitoring_enable                  | true                         | Enable or disable monitoring.                                        |
| firewall_monitoring_email_enable            | false                        | Enable or disable email notifications from periodic.                 |
| firewall_monitoring_email_mailto            | empty                        | Email address to send notifications.                                 |
| firewall_monitoring_email_smarthost         | empty                        | Upstream server for outgoing messages.                               |
| firewall_monitoring_email_port              | 25                           | SMTP server tcp port.                                                |
| firewall_monitoring_email_authpath          | "/etc/dma/auth.conf"         | Path to `auth.conf` for credentials.                                 |
| firewall_monitoring_email_aliases           | "/etc/aliases"               | Path to the local aliases file.                                      |
| firewall_monitoring_email_securetransfer    | true                         | Enable TLS secured transfer.                                         |
| firewall_monitoring_email_starttls          | true                         | Use STARTTLS with securetransfer.                                    |
| firewall_monitoring_email_opportunistic_tls | false                        | Allow STARTTLS to fail. Useful for forwarding without smarthost.     |
| firewall_monitoring_email_secure_login      | true                         | Force TLS for SMTP login.                                            |
| firewall_monitoring_email_mailname          | empty                        | Hostname that `dma` uses to identify the host.                       |
| firewall_monitoring_email_masquerade        | empty                        | Masquerade the  envelope-from  addresses with this address/hostname. |
| firewall_monitoring_email_nullclient        | false                        | Bypass local delivery and forward all mail to smarthost.             |
| firewall_monitoring_email_auth_enable       | false                        | Enable authentication for smarthost.                                 |
| firewall_monitoring_email_auth_username     | empty                        | Username for SMTP authentication.                                    |
| firewall_monitoring_email_auth_password     | empty                        | Password for SMTP authentication.                                    |
| firewall_monitoring_email_forward_root      | false                        | Forward all mail for root to the `mailto` address (uses alias).      |
| firewall_monitoring_vnstat_enable           | false                        | Enable or disable the vnstat service.                                |
| firewall_monitoring_vnstat_config           | "/usr/local/etc/vnstat.conf" | Configuration file location for vnstat.                              |
| firewall_monitoring_vnstat_flags            | "-d"                         | Flags for vnstat on startup.                                         |
| firewall_monitoring_vnstat_default_int      | empty                        | Default interface for vnstat.  Leave blank for auto.                 |
| firewall_monitoring_vnstat_db_path          | "/usr/local/var/lib/vnstat"  | Location of the database for vnstat.                                 |
| firewall_monitoring_vnstat_unitmode         | 0                            | Controls how units are prefixed when traffic is shown.               |
| firewall_monitoring_vnstat_rateunit         | 1                            | Used rate limit. 1 = bits, 0 = bytes.                                |
| firewall_monitoring_vnstat_rateunitmode     | 0                            | How units are prefixed when rate limits are shown.                   |
| firewall_monitoring_vnstat_outputstyle      | 3                            | Output style.  3 = rate column visible.                              |
| firewall_monitoring_vnstat_estbarvisible    | 1                            | Show the estimate bar.                                               |
| firewall_monitoring_vnstat_user             | vnstat                       | User for the vnstat service.                                         |
| firewall_monitoring_vnstat_group            | vnstat                       | Group for the vnstat service.                                        |
| firewall_monitoring_vnstat_bwdetect         | 1                            | Auto detect interface bandwidth.                                     |
| firewall_monitoring_vnstat_maxbw            | 1000                         | Maximum bandwidth (Mbit/s) for all interfaces.                       |
| firewall_monitoring_vnstat_5minute_hours    | 48                           | Data retention for 5 minute intervals.                               |
| firewall_monitoring_vnstat_hourly_days      | 4                            | Data retention for hourly intervals.                                 |
| firewall_monitoring_vnstat_daily_days       | 62                           | Data retention for daily intervals.                                  |
| firewall_monitoring_vnstat_monthly_months   | 25                           | Data retention for monthly intervals.                                |
| firewall_monitoring_vnstat_yearly_years     | -1                           | Data retention for yearly intervals.  (-1 = unlimited)               |
| firewall_monitoring_vnstat_topdayentries    | 20                           | Number of entries for top days.                                      |
| firewall_monitoring_vnstat_updateinterval   | 20                           | How often (in seconds) interface data is updated.                    |
| firewall_monitoring_vnstat_pollinterval     | 5                            | How often (in seconds) interface status changes are checked.         |
| firewall_monitoring_vnstat_saveinterval     | 5                            | How often (in minutes) data is saved to the database.                |
| firewall_monitoring_vnstat_monthrotate      | 1                            | Day of the month to rotate stats.                                    |
| firewall_monitoring_vnstat_uselogging       | 1                            | Enable logging.  0 = disable, 1 = logfile, 2 = syslog.               |
| firewall_monitoring_vnstat_logfile          | "/var/log/vnstat.log"        | Location of the vnstat logfile.                                      |

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
    - wesmarcum.firewall.monitoring
```

Define all variables in the `vars/firewall.yml` file:

```yaml
firewall_monitoring_enable: true
firewall_monitoring_vnstat_enable: true
firewall_monitoring_vnstat_default_int: "em0"
firewall_monitoring_email_enable: true
firewall_monitoring_email_mailto: "email@example.com"
firewall_monitoring_email_smarthost: "smtp.mydomain.com"
firewall_monitoring_email_mailname: "firewall.mydomain.com"
firewall_monitoring_email_masquerade: "noreply@mydomain.com"
```

License
-------

MIT

Author Information
------------------

https://github.com/wesmarcum/
