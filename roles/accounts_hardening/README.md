# accounts_hardening

Hardens local user account settings on Debian servers: password aging policy
(`login.defs`), per-account aging enforcement with exemption support, and a
shell hardening profile (`/etc/profile.d/`) that sets session timeouts, history
hardening, and safety aliases.

## Requirements

- Ansible ≥ 2.20
- Debian 13 (Trixie)
- `become: true`

## Role variables

All variables are defined in `defaults/main.yml` and can be overridden.

### Password aging (`login.defs`)

| Variable | Default | Description |
|---|---|---|
| `shadow_pass_max_days` | `365` | Maximum days a password may be used |
| `shadow_pass_min_days` | `1` | Minimum days between password changes |
| `shadow_pass_warn_age` | `35` | Days of warning before password expiry |

### Session timeout

| Variable | Default | Description |
|---|---|---|
| `session_timeout` | `900` | Idle timeout in seconds (15 min) applied to non-root, non-sudo users |

### Exemptions

| Variable | Default | Description |
|---|---|---|
| `aging_exempt_users` | `[ansible, root]` | Users excluded from password aging enforcement |
| `aging_exempt_groups` | `[teachers]` | Groups whose members are excluded from password aging |

### Hardcoded `login.defs` settings (template, not variables)

- `ENCRYPT_METHOD YESCRYPT` (strongest available hash)
- `UMASK 027`
- `HOME_MODE 0700`
- `LOGIN_RETRIES 3`, `LOGIN_TIMEOUT 60`
- `LOG_OK_LOGINS yes`, `FAILLOG_ENAB yes`

### Shell profile (`/etc/profile.d/`)

- `umask 027` enforced
- `TMOUT` (read-only) for non-root, non-sudo users
- Core dumps disabled (`ulimit -c 0`)
- `noclobber` enabled
- History: 10 000 entries, timestamped, `ignoreboth`, append mode
- Safety aliases: `rm -I`, `cp -i`, `mv -i`

## Tags

| Tag | Tasks |
|---|---|
| `accounts` | All tasks in this role |
| `accounts_login_defs` | `login.defs` shadow password settings |
| `accounts_aging` | Per-account password aging enforcement |
| `accounts_shell` | Shell timeout profile in `/etc/profile.d/` |

## Dependencies

None.

## Example playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: accounts_hardening
      vars:
        shadow_pass_max_days: 180
        shadow_pass_warn_age: 14
        aging_exempt_users:
          - ansible
          - root
          - backup
```

```bash
# Apply with tag filter
ansible-playbook playbooks/security-hardening.yml --tags accounts

# Dry run
ansible-playbook playbooks/security-hardening.yml --tags accounts --check --diff
```

## License

MIT-0
