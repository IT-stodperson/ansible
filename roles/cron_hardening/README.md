cron_hardening
==============

Harden cron on Debian-based servers.
This role restricts cron access to explicitly allowed users via
`/etc/cron.allow`, removes `cron.deny`, and enforces strict ownership
and permissions on cron directories and `/etc/crontab`.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### Access control

| Variable | Default | Description |
|---|---|---|
| `cron_allowed_users` | `[]` | Users allowed to use cron. Root always has access on Debian regardless of this list. |

### Directory and file permissions

| Variable | Default | Description |
|---|---|---|
| `cron_directories` | see `defaults/main.yml` | List of cron directories to enforce `root:root 0700` on |

### What the role hardens (non-variable)

- `/etc/cron.allow` — deployed from template, `root:<crontab group> 0600`
- `/etc/cron.deny` — removed to enforce `cron.allow`-only access
- `/etc/crontab` — enforced `root:root 0600`
- Cron directories (`cron.hourly`, `cron.daily`, `cron.weekly`, `cron.monthly`, `cron.yearly`, `cron.d`) — enforced `root:root 0700`

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all cron hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - cron_hardening
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Allow specific users to use cron

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: cron_hardening
      cron_allowed_users:
        - admin
        - backup
```

### 3. Run only a specific subsystem using tags

Apply only the cron access control (cron.allow/cron.deny):

```bash
ansible-playbook -i inventory site.yml --tags cron_access
```

Apply only the file and directory permission hardening:

```bash
ansible-playbook -i inventory site.yml --tags cron_permissions
```

Available tags: `cron`, `cron_access`, `cron_permissions`.

### 4. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 5. Override variables from the command line

```bash
ansible-playbook -i inventory site.yml \
  -e '{"cron_allowed_users": ["admin", "backup"]}'
```

### 6. Dry-run (check mode)

Preview what the role would change without modifying the system:

```bash
ansible-playbook -i inventory site.yml --check --diff
```

### 7. Use in a larger playbook with other hardening roles

```yaml
- hosts: debian_servers
  become: true
  roles:
    - common
    - cron_hardening
    - apt_hardening
    - pam_hardening
    - auditd
```

### 8. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
cron_allowed_users: []

# group_vars/development.yml
cron_allowed_users:
  - developer
  - ci
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
