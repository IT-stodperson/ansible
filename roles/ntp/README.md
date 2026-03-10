#SPDX-License-Identifier: MIT-0
# ntp

Configure time synchronisation via `systemd-timesyncd` on Debian-based
servers. The role installs the package, creates a drop-in configuration
directory, deploys a hardened `timesyncd.conf` from a Jinja2 template,
and ensures the service is enabled and running.

## Requirements

- Ansible ≥ 2.15
- Debian 12 (Bookworm) targets
- `ansible` SSH user with sudo privileges

## Role variables

There are no user-facing default variables for this role. NTP server
addresses and other timesyncd settings are controlled through the
template at:

```
roles/ntp/templates/60-timesyncd.conf.j2
```

Edit the template directly to change NTP pool servers, fallback NTP,
or other `timesyncd.conf` options.

## Example playbook

```yaml
- hosts: debian_servers
  roles:
    - role: ntp
      tags: ntp
```

## License

MIT-0
