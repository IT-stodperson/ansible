# systemd_hardening

Harden systemd services on Debian-based servers by deploying drop-in
`override.conf` files that restrict capabilities, address families,
namespaces, and filesystem access for each service. The role only applies
overrides to services that are both listed in `systemd_hardening_services`
**and** installed on the target host — skipping services not present.
Services installed but lacking a hardening template emit a non-fatal
warning. A reminder notice is displayed after any override is deployed
to prompt a manual service restart.

## Requirements

- Ansible ≥ 2.20
- Debian 13 (Trixie)
- `become: true`

## Role variables

| Variable | Default | Description |
|---|---|---|
| `systemd_hardening_services` | see below | List of service names (without `.service`) to harden |

Default list:

```yaml
systemd_hardening_services:
  - auditd
  - cron
  - kanidm-unixd
  - ssh
  - systemd-journald
  - systemd-timesyncd
  - wazuh-dashboard
  - wazuh-indexer
  - wazuh-manager
```

## Adding a new service

1. Create a drop-in template at `roles/systemd_hardening/templates/overrides/<service>.conf.j2`
2. Add the service name to `systemd_hardening_services`
3. Re-run the role

## Example playbook

```yaml
- hosts: debian_servers
  roles:
    - role: systemd_hardening
      tags: systemd
```

## License

MIT-0
