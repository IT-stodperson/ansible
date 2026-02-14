apt_hardening
=============

Harden APT (Advanced Package Tool) on Debian-based servers.
This role disables weak dependency installation (Recommends/Suggests),
disables translation downloads, and enforces strict ownership and
permissions on GPG keyrings, source lists, and authentication configs.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### APT configuration

| Variable | Default | Description |
|---|---|---|
| `apt_install_recommends` | `"0"` | Whether to install recommended packages (`0` = disabled) |
| `apt_install_suggests` | `"0"` | Whether to install suggested packages (`0` = disabled) |
| `apt_acquire_languages` | `"none"` | Languages for translation downloads (`none` = disabled) |

### Directory permissions

| Variable | Default | Description |
|---|---|---|
| `apt_directories` | see `defaults/main.yml` | List of APT directories to enforce `root:root 0755` on |

### What the role hardens (non-variable)

- GPG keyring files in `/usr/share/keyrings` and `/etc/apt/trusted.gpg.d` — enforced `root:root 0644`
- Source files in `/etc/apt/sources.list.d` — enforced `root:root 0644`
- Auth files in `/etc/apt/auth.conf.d` — enforced `root:root 0644`
- Verification tasks confirm no files with bad permissions remain after hardening

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all APT hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - apt_hardening
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Override APT configuration settings

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: apt_hardening
      apt_install_recommends: "1"
      apt_acquire_languages: "en"
```

### 3. Run only a specific subsystem using tags

Apply only the APT configuration (weak deps, translations):

```bash
ansible-playbook -i inventory site.yml --tags apt_config
```

Apply only the file and directory permission hardening:

```bash
ansible-playbook -i inventory site.yml --tags apt_permissions
```

Run only the verification checks:

```bash
ansible-playbook -i inventory site.yml --tags apt_verify
```

Available tags: `apt`, `apt_config`, `apt_permissions`, `apt_verify`.

### 4. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 5. Override variables from the command line

```bash
ansible-playbook -i inventory site.yml \
  -e apt_install_recommends=1 \
  -e apt_acquire_languages=en
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
    - apt_hardening
    - accounts_hardening
    - pam_hardening
    - auditd
```

### 8. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
apt_install_recommends: "0"
apt_install_suggests: "0"

# group_vars/development.yml
apt_install_recommends: "1"
apt_install_suggests: "0"
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
