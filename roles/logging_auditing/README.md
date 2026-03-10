# logging_auditing

Hardens logging and auditing on Debian servers: configures journald with
persistent storage and security-focused settings, installs and configures
`auditd` with host-specific audit rules, enables process accounting (`acct`)
and system statistics (`sysstat`).

## Requirements

- Ansible ≥ 2.20
- Debian 13 (Trixie)
- `become: true`

## Role variables

This role has no configurable defaults — all settings are driven by templates:

- `templates/50-journald-hardening.conf.j2` — journald drop-in
- `templates/auditd.conf.j2` — auditd daemon configuration
- `templates/audit-rules/<hostname>.rules.j2` — per-host audit rules

To add audit rules for a host, create
`templates/audit-rules/<inventory_hostname>.rules.j2`.
If no host-specific rules file exists, the base auditd configuration is still
applied but no rules are loaded.

### Post-deployment manual step

After deploying this role, run the following on each target to enable
Forward Secure Sealing (FSS) for journald:

```bash
sudo journalctl --setup-keys
```

Save the verification key in your password manager.

## Tags

| Tag | Tasks |
|---|---|
| `logging` | All tasks in this role |

## Dependencies

None.

## Example playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: logging_auditing
```

```bash
ansible-playbook playbooks/security-hardening.yml --tags logging
```

## License

MIT-0
