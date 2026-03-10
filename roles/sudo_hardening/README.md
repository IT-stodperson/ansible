# sudo_hardening

Harden sudo on Debian-based servers. The role deploys a secure sudoers
drop-in configuration with session time limits, full I/O session logging,
a restricted `secure_path`, alert email on misuse, and hardened umask.
It audits all sudoers files for `NOPASSWD` entries and fails if unexpected
ones are found. File and directory permissions on `/etc/sudoers` and
`/etc/sudoers.d/` are enforced to prevent privilege escalation.

## Requirements

- Ansible ≥ 2.20
- Debian 13 (Trixie)
- `become: true`

## Role variables

### Environment and timing

| Variable | Default | Description |
|---|---|---|
| `sudo_timestamp_timeout` | `15` | Minutes of inactivity before sudo requires a password again |
| `sudo_passwd_timeout` | `1` | Minutes allowed to enter password before prompt times out |
| `sudo_timestamp_type` | `"global"` | Timestamp scope: `global` (once per session) or `tty` (per terminal) |

### Logging

| Variable | Default | Description |
|---|---|---|
| `sudo_logfile` | `"/var/log/sudo.log"` | Dedicated log file for all sudo commands |
| `sudo_iolog_dir` | `"/var/log/sudo-io"` | Directory for I/O session logs |

### Security

| Variable | Default | Description |
|---|---|---|
| `sudo_umask` | `"0077"` | umask applied to processes launched by sudo |
| `sudo_secure_path` | `"/usr/local/sbin:..."` | Restricted `PATH` for sudo-launched processes |
| `sudo_passwd_tries` | `3` | Maximum password attempts before sudo fails |

### Alerts

| Variable | Default | Description |
|---|---|---|
| `sudo_mailto` | `"itstodperson@gymnasium.ax"` | Email address for sudo security alerts |

### Allowed groups

| Variable | Default | Description |
|---|---|---|
| `sudo_allowed_groups` | `["sudo"]` | Unix groups granted full sudo access |

### Service accounts

| Variable | Default | Description |
|---|---|---|
| `sudo_service_accounts` | see below | List of accounts that require passwordless sudo |

Default entry:

```yaml
sudo_service_accounts:
  - name: ansible
    privileges: "ALL=(ALL) NOPASSWD: ALL"
    comment: Ansible automation — requires passwordless sudo for remote management
```

Each item requires `name`, `privileges`, and `comment`. The `comment`
field is written as a sudoers comment above the rule.

## Example playbook

```yaml
- hosts: debian_servers
  roles:
    - role: sudo_hardening
      tags: sudo
```

## License

MIT-0
