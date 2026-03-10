# cron_hardening

Hardens `cron` on Debian servers: restricts access to an explicit allowlist via
`/etc/cron.allow`, and enforces `root:root 0700` permissions on all cron
directories and `0600` on `/etc/crontab`.

## Requirements

- Ansible ≥ 2.20
- Debian 13 (Trixie)
- `become: true`

## Role variables

All variables are defined in `defaults/main.yml` and can be overridden.

| Variable | Default | Description |
|---|---|---|
| `cron_allowed_users` | `[]` | Users allowed to use cron. Root always has access. An empty list means only root can use cron. |
| `cron_directories` | `/etc/cron.{hourly,daily,weekly,monthly,yearly,d}` | Directories to enforce `root:root 0700` on |

## Tags

| Tag | Tasks |
|---|---|
| `cron` | All tasks in this role |
| `cron_access` | Access control (`/etc/cron.allow`) |
| `cron_permissions` | Directory and file permission hardening |

## Dependencies

None.

## Example playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: cron_hardening
      vars:
        cron_allowed_users:
          - backup
```

```bash
ansible-playbook playbooks/security-hardening.yml --tags cron
```

## License

MIT-0
