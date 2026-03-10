# motd

Deploy a dynamic Message-of-the-Day on Debian-based servers. The role
deploys a shared base MOTD script (system info, uptime, disk/memory usage,
last login) and automatically includes per-host snippet templates when
found under `templates/scripts/per_host/<hostname>_*.j2`. A systemd
cache service/timer pre-generates slow data at boot and every six hours;
an APT hook refreshes the cache after each package update.

## Requirements

- Ansible ≥ 2.20
- Debian 13 (Trixie)
- `become: true`

## Role variables

| Variable | Default | Description |
|---|---|---|
| `motd_cache_timer_boot_delay` | `"2min"` | Delay after boot before the cache service first runs |
| `motd_cache_timer_interval` | `"6h"` | How often the cache timer fires to refresh slow MOTD data |
| `motd_role` | `"Server"` | Server role label shown in the MOTD header — override per host |

Override `motd_role` in `host_vars/<hostname>/vars.yml` to give each server
a meaningful label (e.g. `"Wazuh SIEM"`, `"Ansible Controller"`).

## Tags

| Tag | What it runs |
|---|---|
| `motd` | All MOTD tasks |
| `motd_cache` | Cache infrastructure only (service, timer, APT hook) |
| `motd_script` | MOTD script deployment only |

## Per-host snippets

Create a Jinja2 template at:

```
roles/motd/templates/scripts/per_host/<inventory_hostname>_<section>.j2
```

The base script includes all matching snippets automatically via
`{% include ... ignore missing %}`. No task changes are needed — just
add the template file and re-run the role.

## Example playbook

```yaml
- hosts: debian_servers
  roles:
    - role: motd
      tags: motd
```

## License

MIT-0
