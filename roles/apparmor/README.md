apparmor
========

Harden AppArmor on Debian-based servers. This role installs AppArmor
and its utilities, ensures the service is enabled and running, enforces
all profiles in `/etc/apparmor.d/`, and validates that AppArmor is not
disabled via GRUB boot parameters.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### Profile enforcement

| Variable | Default | Description |
|---|---|---|
| `apparmor_enforce_profiles` | `true` | Whether to enforce all AppArmor profiles found in `apparmor_profile_path` |
| `apparmor_profile_path` | `"/etc/apparmor.d/*"` | Path glob passed to `aa-enforce` |

### GRUB boot parameter validation

| Variable | Default | Description |
|---|---|---|
| `apparmor_grub_check` | `true` | Whether to check GRUB for `apparmor=0` and fail if found |

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all AppArmor hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - apparmor
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Override specific variables

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: apparmor
      apparmor_enforce_profiles: true
      apparmor_profile_path: "/etc/apparmor.d/usr.*"
      apparmor_grub_check: false
```

### 3. Run only a specific subsystem using tags

Install AppArmor packages and enable the service only:

```bash
ansible-playbook -i inventory site.yml --tags apparmor_install
```

Enforce profiles only (skip install and GRUB check):

```bash
ansible-playbook -i inventory site.yml --tags apparmor_enforce
```

Run only the GRUB boot parameter validation:

```bash
ansible-playbook -i inventory site.yml --tags apparmor_grub
```

Available tags: `apparmor`, `apparmor_install`, `apparmor_enforce`,
`apparmor_grub`.

### 4. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 5. Override variables from the command line

```bash
ansible-playbook -i inventory site.yml \
  -e apparmor_enforce_profiles=false \
  -e apparmor_grub_check=false
```

### 6. Dry-run (check mode)

Preview changes without modifying the system:

```bash
ansible-playbook -i inventory site.yml --check --diff
```

### 7. Use in a larger playbook with other hardening roles

```yaml
- hosts: debian_servers
  become: true
  roles:
    - common
    - accounts_hardening
    - pam_hardening
    - apparmor
    - auditd
```

### 8. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
apparmor_enforce_profiles: true
apparmor_grub_check: true

# group_vars/development.yml
apparmor_enforce_profiles: false
apparmor_grub_check: false
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
