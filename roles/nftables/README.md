nftables
========

Manage nftables firewall on Debian-based servers. This role provides
the framework for deploying a base nftables ruleset and host-specific
firewall rules.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

This role has no configurable default variables. Firewall rules are
managed through templates and host-specific configuration.

Dependencies
------------

None.

Use Cases
---------

### 1. Apply nftables configuration with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - nftables
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
    - nftables
    - ssh_hardening
    - logging_auditing
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
