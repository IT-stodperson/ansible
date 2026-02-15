ntp
===

Configure NTP via systemd-timesyncd on Debian-based servers. This role
deploys a secure timesyncd configuration with trusted NTP sources and
ensures the service is enabled and running.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

This role has no configurable default variables. NTP server
configuration is managed through the template
`templates/60-timesyncd.conf.j2`.

### What the role configures (non-variable)

- `/etc/systemd/timesyncd.conf.d/60-timesyncd.conf` — NTP server
  configuration deployed from template
- `systemd-timesyncd` service — enabled and started

Dependencies
------------

None.

Use Cases
---------

### 1. Apply NTP configuration with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - ntp
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
    - ntp
    - kernel_hardening
    - accounts_hardening
    - ssh_hardening
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
