# ssh_hardening

Harden SSH on Debian-based servers. The role filters weak Diffie-Hellman
moduli from `/etc/ssh/moduli` (keeping only those ≥ the configured bit
threshold), deploys a secure `sshd_config` drop-in that restricts
algorithms, disables root login, and limits SSH access to specified Unix
groups. The `sshd` service is reloaded on any configuration change.

## Requirements

- Ansible ≥ 2.20
- Debian 13 (Trixie)
- `become: true`

## Role variables

| Variable | Default | Description |
|---|---|---|
| `ssh_moduli_min_bits` | `3071` | Minimum DH modulus size to keep in `/etc/ssh/moduli`; weaker moduli are removed |
| `ssh_allow_groups` | `["teachers", "ansible"]` | Unix groups permitted to connect via SSH (`AllowGroups` directive) |

Override `ssh_allow_groups` in `group_vars/` or `host_vars/` to adjust
which groups are granted SSH access per host or host group.

## Example playbook

```yaml
- hosts: debian_servers
  roles:
    - role: ssh_hardening
      tags: ssh
```

## License

MIT-0
