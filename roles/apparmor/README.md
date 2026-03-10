# apparmor

Installs AppArmor and its utilities, ensures the service is running, validates
and enforces all profiles in `/etc/apparmor.d/`, optionally downgrades specific
profiles to complain mode, unloads unwanted profiles from the running kernel,
and asserts that AppArmor has not been disabled via GRUB boot parameters.

## Requirements

- Ansible ≥ 2.20
- Debian 13 (Trixie)
- `become: true`

## Role variables

All variables are defined in `defaults/main.yml` and can be overridden.

| Variable | Default | Description |
|---|---|---|
| `apparmor_complain_profiles` | `[]` | Profiles to set to complain mode after the enforce pass (log but don't block) |
| `apparmor_remove_profiles` | `[]` | Profiles to persistently disable and unload from the kernel (e.g. `runc` on kernel 6.12+ with Docker) |

## Tags

This role does not define sub-task tags. Apply the whole role with:

```bash
ansible-playbook playbooks/security-hardening.yml --tags apparmor
```

## Dependencies

None.

## Example playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: apparmor
```

### Set specific profiles to complain mode

Useful when a profile is too restrictive for a particular service but you still
want to log its activity:

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: apparmor
      vars:
        apparmor_complain_profiles:
          - usr.sbin.rsyslogd
```

### Remove a profile that breaks a service

```yaml
- hosts: docker_hosts
  become: true
  roles:
    - role: apparmor
      vars:
        apparmor_remove_profiles:
          - runc
```

```bash
# Dry run
ansible-playbook playbooks/security-hardening.yml --tags apparmor --check --diff
```

## License

MIT-0
