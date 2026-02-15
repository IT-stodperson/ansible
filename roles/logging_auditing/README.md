logging_auditing
================

Harden logging and auditing on Debian-based servers. This role configures
journald with persistent storage and security-focused settings, installs
and configures auditd with host-specific audit rules, and ensures both
services are enabled and running.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).
- Host-specific audit rules templates must be placed in
  `templates/audit-rules/<inventory_hostname>.rules.j2` for each server.

Role Variables
--------------

This role has no configurable default variables. All configuration is
managed through templates.

### What the role configures (non-variable)

**Journald** (`/etc/systemd/journald.conf.d/50-security-hardening.conf`):
- Persistent storage, compression, rate limiting, Forward Secure Sealing

**Auditd** (`/etc/audit/auditd.conf`):
- Daemon configuration (log format, buffer size, retention)

**Audit rules** (`/etc/audit/rules.d/audit.rules`):
- Host-specific rules deployed from `templates/audit-rules/<hostname>.rules.j2`
- If no host-specific template exists, the role skips rule deployment and
  warns instead

### Post-deployment manual step

After the role runs, you must generate Forward Secure Sealing keys on
each host:

```bash
sudo journalctl --setup-keys
```

Save the verification key in your password manager.

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all logging and auditing hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - logging_auditing
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 3. Dry-run (check mode)

Preview changes without modifying the system:

```bash
ansible-playbook -i inventory site.yml --check --diff
```

### 4. Use in a larger playbook with other hardening roles

```yaml
- hosts: debian_servers
  become: true
  roles:
    - kernel_hardening
    - accounts_hardening
    - pam_hardening
    - logging_auditing
    - ssh_hardening
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
